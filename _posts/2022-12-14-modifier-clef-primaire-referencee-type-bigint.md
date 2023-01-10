---
layout: post
title:  "Passer une PK référencée du type INT en BIGINT"
date:   2022-12-14 16:59:42 +0200
categories: bdd
excerpt: "Sur plusieurs tables volumineuses **avec** PostgreSQL"
cover:
  image: "change-referenced-pk-bigint.jpg"
  source_title: How to find foreign keys references
  source_url: https://simplesqltutorials.com/sql-server-find-foreign-key-references/
authors: pierre_top
---

## TL;DR

Dans un [précédent article]({% post_url 2021-08-31-modifier-clef-primaire-non-referencee-type-bigint %}), nous partagions comment modifier une propriété du type `INT` vers le type `BIGINT`:

- si celle-ci n'était pas référencée par une autre table;
- en minimisant le temps d'indisponibilité;
- en modifiant les données dans une colonne temporaire de la table.

Depuis, nous avons modifié une propriété référencée, en l'occurrence par une clef étrangère.
Nous avons essayé de migrer les données progressivement sur une copie de la table:

- pour limiter la fragmentation;
- pour ne pas modifier l'organisation des données;
- en modifiant l'allocation des ressources CPU/mémoire de la migration.

Nous avons fini par comprendre que le facteur limitant était l'I/O, et que celle-ci, contrôlée par notre PaaS, ne pouvait être améliorée.

Nous avons abandonné cette piste et utilisé un créneau de maintenance d'une journée pour migrer en utilisant la solution native.

Nous avons surtout retenu de consulter le métier le plus tôt possible (ici pour se mettre d'accord sur les plages de maintenance planifiées), afin
de faire les choix techniques les mieux informés.

## Motivation

La [motivation reste la même]({% post_url 2021-08-31-modifier-clef-primaire-non-referencee-type-bigint %}#motivation) que sur la propriété non référencée, pour rappel:

- pas d'interruption de service;
- pas de dégradation des temps de réponse pendant l'ensemble des opérations.

A l'arrivée, on souhaite que ces 3 éléments soient de type `BIGINT`:

- la propriété elle-même : `id` sur la table `answers`;
- la séquence utilisée pour alimenter cette propriété : `answers_id_seq`;
- les propriétés référençant cette propriété : clef étrangère `answerId` sur la table `knowledge-elements`.

## Recherche de solution

### Solution 1 : instructions natives PostgreSQL

Nous avons commencé par obtenir le temps d'exécution de la solution native sur un environnement comparable à l'environnement cible.
Il servira de référence pour le temps maximal d'indisponibilité: si l'on trouve une solution avec un temps inférieur à la solution native, elle est retenue comme candidate.

La solution native se résume à ces instructions, et prend 15h.

```postgres-sql
ALTER TABLE "knowledge-elements" ALTER COLUMN "answerId" TYPE BIGINT;
ALTER TABLE "flash-assessment-results" ALTER COLUMN "answerId" TYPE BIGINT;
ALTER TABLE "answers" ALTER COLUMN "id" TYPE BIGINT;
ALTER SEQUENCE AS BIGINT;
```

Le temps d'exécution est partagé de manière à peu près égale entre les deux tables, et celles-ci sont successivement inaccessibles en lecture et en écriture.
Ces tables étant utilisées de manière conjointe, cela donne un temps d'indisponibilité de 15 heures.

### Solution 2 : migration en arrière plan

#### Investigation

Les données à modifier sont:

- les méta-données de la table, utilisées lors de la création d'un enregistrement;
- les données existantes brutes, c'est à dire les enregistrements de table.

Cependant, les données d'index doivent aussi être modifiées explicitement dans cette solution alors que ces modifications sont implicites dans les instructions natives.
Nous y reviendrons en détail dans la seconde partie.

##### Migration des données

###### Migration dans la même table

Comme dans [l'article précédent]({% post_url 2021-08-31-modifier-clef-primaire-non-referencee-type-bigint %}#implémentation), le temps d'indisponibilité ne peut être réduit qu'à condition d'effectuer:

- tout d'abord la plus grande partie du travail, la migration des données existantes, en tâche de fond;
- et ensuite de migrer les données récentes lors d'un créneau de maintenance réduit.

Si la migration en tâche de fond est effectuée sur la même instance de base de données, ou la même table, elle peut impacter l'activité habituelle.

C'est pour cela que nous commençons par [un POC][pull-request-custom-v1], qui migre les données:

- par batch;
- avec un délai d'exécution;
- pouvant être interrompu et relancé sans recommencer toute la migration;
- avec des réglages (ex: taille de batch) modifiables en temps réel.

Devant cette complexité, qui nous expose à la corruption de données, nous choisissons d'implémenter:

- la migration en NodeJs, afin de disposer de tests automatisés;
- la copie des données insérées entre-temps (qui ne peut être faite que coté serveur) via des triggers PostgreSQL.

Le dispositif est le suivant:

- une migration de BDD crée:
  - la colonne de type BIGINT;
  - la table de paramétrage;
  - les triggers de copie ;
- un script client migre les données existantes:
  - il lit le paramétrage dans une table;
  - il migre un batch de données et trace les données traitées;
  - il relit le paramétrage au cas où il aurait été mis à jour;
  - il traite le prochain batch là où il s'était arrêté.

Le code client a été réalisé:

- en TDD, avec des tests unitaires/intégration/acceptance (700 lignes sur les 1000 de la pull-request);
- en Clean Architecture (voir le [use-case][pull-request-custom-v1:use-case]).

###### Obstacle: la fragmentation

Il y a un compromis à trouver en ce qui concerne la taille des batchs:

- plus le nombre de lignes traité par batch est petit, plus l'impact est réduit sur les autres requêtes;
- plus le nombre de lignes traité par batch est grand, plus l'exécution de la migration est rapide.

Lors de la migration sur la table non référencée, nous avions effectué des batchs de 1 million d'enregistrements, pour
une durée moyenne de 2 minutes, en gardant une expérience utilisateur dégradée mais acceptable (pc98 de l'ordre de 100 ms => 1 s).
Il est maintenant possible de baisser la taille de ces batchs à 100 000 enregistrements pour moins impacter le temps de réponse.
Mais il faut prendre en compte un autre facteur: la fragmentation des données dans la table.

Lorsque PostgreSQL met à jour l'enregistrement (`UPDATE answers SET id_bigint = id`), il ne met pas à jour l'original, mais insère un autre
enregistrement et marque le précédent comme obsolète (c'est le [MVCC][]). De plus, l'insertion de données se fait par ensemble d'enregistrements (page)
pour des questions de performance. Sans oublier que les données normalement insérées par l'application continuent d'arriver.

Ici, nous demandons la modification de tous les enregistrements de la table: PostgreSQL va progressivement réécrire sur le disque l'intégralité de
la table. Il est possible qu'à la fin de cette opération, les données:

- soient physiquement stockées dans un ordre différent que celui dans lequel elles ont été écrites initialement;
- prennent plus de place.

Après la migration sur la table non référencée, nous avons constaté:

- une densité d'enregistrements par page qui varie de 60 à 35;
- sur des pages contigües, des données d'octobre 2020 côtoient des données de février 2021, puis l'on repasse à octobre 2020.

Cela n'est pas forcément un problème:

- si vos index sont suffisamment sélectifs;
- si vos données ne sont pas interrogées chronologiquement.

Dans notre cas, la perspective d'obtenir des résultats encore plus accentués en diminuant la taille des batchs ne nous convenait pas.

###### Migration des données dans une table séparée

####### Exploration

Prenons du recul sur le but: migrer les données en BIGINT sans modifier l'activité sur la table cible.

Comme application ne fait qu'insérer des données dans la table, nous décidons:

- d'isoler les données existantes dans une nouvelle table;
- de les transformer;
- dans le créneau de maintenance, de rajouter les données insérées au fil de l'eau;
- de remplacer la table d'origine par la nouvelle table.

Lorsque se pose la question d'alimentation de la nouvelle table, nous nous rendons compte qu'il est possible de transformer les données en même temps.
Si la propriété de la nouvelle table est de type BIGINT, PostgreSQL effectue une coercition de type (cast) sur la donnée INT.

```postgres-sql
CREATE TABLE "answers_bigint" (id BIGINT,..);
INSERT INTO "answers_bigint" (id, ..)
SELECT id, ... FROM answers;
```

Nous avons aussi envisagé deux variantes:

- pour décharger la BDD de la lecture des données, il est possible d'importer un [dump][poc-sorted-table-creation-dump];
- pour insérer les données dans un ordre particulier, en délégant le tri à [un processus hors BDD][poc-sorted-table-creation].

Nous sommes restés sur la solution `INSERT INTO` :

- l'impact sur la BDD était négligeable;
- elle est la plus rapide;
- nous n'étions pas sûr du tri qui nous serait le plus bénéfique;
- elle était la plus simple.

####### Implémentation

Comme on travaille avec deux tables, pour garder l'intégrité des données, nous choisissons d'implémenter:

- la migration en NodeJs, afin de disposer de tests automatisés;
- la copie des données insérées entre-temps (qui ne peut être faite que coté serveur) via des triggers PostgreSQL.

Le tout se compose de 6 pull-requests, afin de faciliter les revues:

- [partie 1][pull-request-custom-v2-part1]: création des tables cibles;
- [partie 2][pull-request-custom-v2-part2]: copie des données;
- [partie 3][pull-request-custom-v2-part3]: création des index;
- [partie 4][pull-request-custom-v2-part4]: recopie des données au fil de l'eau;
- [partie 5][pull-request-custom-v2-part5]: récupération des données non recopiés au fil de l'eau;
- [partie 6][pull-request-custom-v2-part6]: exposition des tables cibles.

##### Création des index

Dans l'implémentation précédente, vous avez peut-être remarqué que la création des index était effectué après la copie des données.

Nous avons fait ce choix pour deux raisons:

- intuitivement, la création d'index en une seule fois, qui requiert la lecture [toute la table][index-explain-plan], semble plus efficace que des mises à jour d'index à chaque insertion d'enregistrement;
- cela permet de contrôler les ressources CPU allouées.

Une partie de la création d'index peut être effectuée en parallèle par des agents (`worker`) sur des CPU différents.

[La documentation][doc-pg-index-parallel] explique les pré-requis de cette allocation.
> PostgreSQL can build indexes while leveraging multiple CPUs in order to process the table rows faster for index methods that support building indexes in parallel (currently, only B-tree).
>
> (..)
>
> Generally, a cost model automatically determines how many worker processes should be requested, if any.
>
> (..)
>
> Increasing max_parallel_maintenance_workers may allow more workers to be used, which will reduce the time needed for index creation, so long as the index build is not already I/O bound.
> Of course, there should also be sufficient CPU capacity that would otherwise lie idle.

[Elle ajoute][doc-pg-worker] mentionne:
> Parallel workers are taken from the pool of processes established by max_worker_processes, limited by max_parallel_workers.
> Note that the requested number of workers may not actually be available at run time.
> If this occurs, the utility operation will run with fewer workers than expected.
>
>(..)
>
>Note that parallel utility commands should not consume substantially more memory than equivalent non-parallel operations.
>
>(..)
>
>However, parallel utility commands may still consume substantially more CPU resources and I/O bandwidth.

###### Allouer peu de ressources

Nous souhaitons allouer un seul agent pour créer cet index, pour laisser la majorité des ressources disponibles à l'application.

Nous essayons d'abord de modifier ce paramètre pour toute la base de données.
Cela affecte les création d'index et les `VACUUM`.

Il vaut par défaut 2, nous le passons au minimum: 1.

```postgres-sql
SHOW max_parallel_maintenance_workers;
> max_parallel_maintenance_workers
>----------------------------------
> 2
SET max_parallel_maintenance_workers = 1;
```

Nous démarrons des simulations d'activité sur l'application et démarrons la création d'index.
On observe une dégradation nette des temps de réponse.

Nous essayons de limiter cette modification uniquement à cette table.

```postgres-sql
ALTER TABLE answers_bigint  SET (parallel_workers = 1);
SELECT reloptions FROM pg_class WHERE oid = 'answers_bigint'::regclass::oid;
>      reloptions
>----------------------
> {parallel_workers=1}
```

On observe toujours une dégradation.

###### Allouer beaucoup de ressources

L'hypothèse de ne pas impacter les temps de réponse semble compromise.
Nous nous dirigeons alors vers un créneau de maintenance, où l'on essaierait de créer l'index le plus rapidement possible.
Pour cela, il nous faut donner la majorité des ressources. Nous augmentons le nombre de worker pour qu'il soit le même que le nombre de CPU.

Nous suivons le nombre de worker réellement utilisés.

```postgres-sql
SELECT
    current_setting('max_parallel_workers')::integer AS max_workers,
    COUNT(*) AS active_workers
FROM pg_stat_activity
WHERE backend_type = 'parallel worker';
```

Malgré cela, le temps d'exécution ne change pas.

Nous explorons [d'autres pistes][doc-cybertec-index-parallel], comme la mémoire ou les tablespace alloués.
Nous ne constatons pas de changement de comportement.

###### Utiliser l'allocation native

Notre dernier essai est d'abandonner la création d'index en parallèle, pour tester cet extrait de la documentation.
> However, parallel utility commands may still consume substantially more CPU resources and I/O bandwidth.

La durée d'exécution est la même, 12 heures, un peu moins que la solution native, pour une complexité bien supérieure.

```text
Table answers
- PK: 35 minutes
- FK: 21 minutes
- NOT NULL: 2 heures 16 minutes
- Autres index: 31 minutes

Table knowledge-elements
- PK: 1 heure 17 minutes
- FK: 2 heures 50 minutes
- NOT NULL: 3 heures 28 minutes
- Autres index: 1 heure 8 minutes
```

###### Le temps de la réflexion

Nous avons testé beaucoup de réglages et prenons un peu de recul, à savoir relire la documentation. Cette phrase attire enfin notre attention.
> Increasing max_parallel_maintenance_workers may allow more workers to be used, which will reduce the time needed for index creation,
> **so long as the index build is not already I/O bound**.

Nous faisons [appel à la communauté][post-stack-exchange-index-creation] pour savoir comment vérifier si l'I/O est le facteur limitant.
La réponse est de nous tourner vers [notre PaaS](https://scalingo.com/), car ces métriques ne sont pas accessibles dans la base de données.
C'est ce que nous faisons, et la réponse est claire

> D'après nos hypothèses, la limitation ici serait lié aux IOPS. Pourquoi ? Lors de vos opérations intenses en lectures/écritures, vous pouvez voir que le CPU ne monte pas tellement
> considérant qu'ils y a 8 cores disponibles avec 4 garanties (400% CPU) pour le plan. Le nombre d'IOPS disponible est proportionnel à la capacité de la base (..) donc 4500 IOPS.
>
> Les IOPS dans l'univers du cloud est souvent ce qui est le plus coûteux par rapport à la capacité et nous ne pouvons pas simplement vous donner 20 000 IOPS par exemple.

#### Bilan

La solution native prenait 15 heures et la solution en arrière plan nécessitait avec un créneau de maintenance de 12 heures.
Lors d'essais quelques mois après, la durée d'exécution de la solution en arrière plan prenait encore plus de temps, alors que la solution native restait stable.

Il restait la possibilité de répartir la création des index dans le temps, dans les plages de faible activité de la plateforme (nuit, week-end).
Cela entrainait malgré tout une baisse de la qualité de service pendant la création d'au moins un index, qui pouvait prendre jusqu'à 2h50 pour le plus important.
Il serait dans ce cas nécessaire de mobiliser une équipe pendant ce créneau, pour monitorer les opérations et les interrompre si la qualité de service était trop impactée.

Nous avons alors proposé ces deux solutions, native et en arrière plan, à l'équipe métier, au contact des utilisateurs et ayant une vision d'ensemble.
Leur réponse fut claire; ils préféraient une solution simple, avec interruption totale de service, prévisible et annoncée à l'avance aux partenaires et aux utilisateurs.

Nous avons donc choisi la solution native, pour sa constance en termes de temps d'exécution et sa faible complexité.
Cela a été l'occasion de multiples apprentissages, notamment du risque de réorganisation de la table et des facteurs limitant sur des opérations intensives.

Il existe peut-être une solution sans interruption de service, par exemple dans une instance séparée et une réplication logique, mais ces services ne sont pas disponibles pour l'instant.

## Implémentation

La solution native est dans [cette courte pull-request][pull-request-native].

## Exécution en production et perspectives

Nous avions appris dans la phase d'exploration que la durée d'exécution assez longue de la migration sans état d'avancement était frustrant.
L'exécution de la solution native étant prévue sur un créneau de faible fréquentation, à savoir la nuit, il était important de disposer d'informations
claires sur ce qui se passait pour pouvoir prendre les bonnes décisions, même sous la fatigue et la pression.

Un [monitoring][migration-monitoring-post] complet a été mis en place sur les 3 phases:

- arrêt des applications;
- avancement de la migration;
- redémarrage des applications.

L'exécution en production s'est bien passée. La durée de traitement a été plus longue que sur l'environnement de test,
à savoir 19 heures au lieu de 12 heures. Nous ne connaissons pas la cause de cet allongement; on peut faire l'hypothèse
que les données étaient éparpillées sur le disque en production alors que sur la plateforme de test, importée par dump,
les données étaient contigües.

Nous en retenons la nécessité de prévoir un créneau de maintenance un peu plus important.

[previous-migration-post#implementation]: https://engineering.pix.fr/bdd/2021/09/03/modifier-clef-primaire-non-referencee-type-bigint.html#implementation
[pull-request-native]: https://github.com/1024pix/pix/pull/3839
[pull-request-custom-v1]: https://github.com/1024pix/pix/pull/3817
[pull-request-custom-v1:use-case]: https://github.com/1024pix/pix/pull/3817/files#diff-a0caa09715e2b8ff7e9d0e7bdd7b7f072862d3af9d290c0314a827467702e55b
[poc-sorted-table-creation]: https://github.com/1024pix/pix/commit/07200e0ed9677819d7ac0d6c335ca1daf0e94e95
[poc-sorted-table-creation-dump]: https://github.com/1024pix/pix/commit/7aefd223c2340e8533a680a2507ac14ca3b739ec#diff-76a491eba3a01404c3a05b940c2101e722f17a27fb81d390bdc9b33cf080a645
[pull-request-custom-v2-part1]: https://github.com/1024pix/pix/pull/4108
[pull-request-custom-v2-part2]: https://github.com/1024pix/pix/pull/4127
[pull-request-custom-v2-part3]: https://github.com/1024pix/pix/pull/4130
[pull-request-custom-v2-part4]: https://github.com/1024pix/pix/pull/4134
[pull-request-custom-v2-part5]: https://github.com/1024pix/pix/pull/4135
[pull-request-custom-v2-part6]: https://github.com/1024pix/pix/pull/4137
[index-explain-plan]: https://stackoverflow.com/questions/43484346/explain-analyze-on-create-index-with-postgresql
[doc-pg-index-parallel]: https://www.postgresql.org/docs/current/sql-createindex.html#Notes
[doc-pg-worker]: https://www.postgresql.org/docs/13/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS-MAINTENANCE
[doc-cybertec-index-parallel]:https://www.cybertec-postgresql.com/en/postgresql-parallel-create-index-for-better-performance/
[post-stack-exchange-index-creation]: https://dba.stackexchange.com/questions/305356/track-down-which-resource-limits-index-creation-in-postgresql
[migration-monitoring-post]: https://engineering.pix.fr/bdd/2022/09/20/steampipe-dashboard.html
[MVCC]: https://www.postgresql.org/docs/current/mvcc.html
