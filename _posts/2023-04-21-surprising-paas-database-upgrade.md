---
layout: post
title:  "Une montée de version PostgreSQL en PaaS surprenante"
date:   2023-04-21 14:27:00 +0200
categories: bdd
excerpt: "Les contraintes ne sont pas toujours respectées.."
cover:
  image: "surprising-paas-database-upgrade.jpg"
  source_title: Un véritable doublon, une pièce d'or espagnole (Wikipedia)
  source_url: https://en.wikipedia.org/wiki/Doubloon#/media/File:Doubloon.jpg
authors: pierre_top
---

## TL;DR

Les Database-as-a-service, c'est pratique pour ne pas avoir trop d'expertise en interne. Mais parfois, on a quand même des surprises.

Si vous :
- êtes hébergés par Scalingo;
- utilisez les addons PostgreSQL;
- souhaitez passer de la version 13.7 à 13.9.

Alors prévoyez un créneau de maintenance pour recréer certains index.

Pourquoi ?
- parce qu'une mise à jour d'addon inclut, en plus de l'application "base de données", potentiellement, des mises à jour de l'OS ;
- un lien existe entre l'OS et les index qui utilisent [une collation](https://www.postgresql.org/docs/15/collation.html) par défaut.


## Le décor

Pix utilise un PaaS, [Scalingo](https://scalingo.com/), pour l'hébergement de la plupart ses applications.
Il fournit un service de base de données sous la forme d'addon, dont la gestion (sauvegarde, PITR, réplication, maintenance) est garantie.
Nous utilisons ce service, car notre utilisation ne justifie pas la formation d'une équipe dédiée en interne.

Nous gardons toutefois la main sur le déclenchement des montées de version, car celles-ci requièrent des interruptions de service [dans certains cas](https://doc.scalingo.com/databases/postgresql/start#database-upgrade).
> If your database uses a business plan, we are able to achieve zero-downtime upgrade of minor version.
> In the case of major version upgrade, we need to completely stop the nodes, hence we can’t achieve zero-downtime.

Nous effectuons préalablement des tests automatisés:

- de non-régression sur la version de BDD [en local et dans la CI](https://github.com/1024pix/pix/pull/5431);
- de performance sur l'environnement cible Scalingo.

A la sortie de PostgreSQL 14 sur Scalingo [en janvier](https://scalingo.com/blog/dbaas-postgresql14), nous avons déroulé notre scénario habituel.

## Le drame

Le lendemain de la mise à jour en production, le monitoring remonte des erreurs 500 sur plusieurs routes API.
Les erreurs n'étant pas nombreuses et limitées à l'authentification, nous analysons les logs de manière détaillée..
et nous rendons compte qu'il y a deux comptes actifs possédant le même email. Cela nous surprend, puisque nous nous sommes assurés que cela n'arriverait pas, grâce au mécanisme de [contrainte d'unicité](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-UNIQUE-CONSTRAINTS)
en base de données qui exclut cette possibilité.

Fait troublant, les logs mentionnent aussi des requêtes de mise à jour de données rejetées pour cause de violations de contraintes.

Une première requête nous rassure.

```sql
SELECT id, email FROM users WHERE email = 'john.doe@example.net';
id  | email
----------
1   | john.doe@example.net
(1 row)
```

Mais une deuxième nous inquiète.

```sql
SELECT id, email FROM users WHERE id IN (1,666);
id  | email
----------
1   | john.doe@example.net
666 | john.doe@example.net
(2 rows)
```

Force est de constater qu'il y a plusieurs enregistrements :

- avec des identifiants différents ;
- et un champ email identique.

Il existe pourtant [une contrainte UNIQUE](https://github.com/1024pix/pix/blob/dev/api/db/migrations/20170418114929_remove_login_from_users.js).

```sql
\d users
Indexes:
"users_email_unique" UNIQUE CONSTRAINT, btree (email)
```

Nous ne savons pas pourquoi cela se produit, mais nous avons accepté le fait qu'il y ait des doublons.
Comme nous n'arrivons pas à récupérer tous les enregistrements en doublons (probablement une optimisation basée sur la contrainte d'unicité),
nous revenons à une détection bas-niveau avec un hash sur le champ en question.

```sql
SELECT md5(email) AS hash,
COUNT(*) FROM users
HAVING COUNT(*) > 1;

hash          | count(*)
------------------------------------
40c05(..)     | 2
```

## L'enquête

Nous sommes parti d'un seul indice : nous avions mis à jour la base de données la veille.
Notre hypothèse est que la mise à jour de PostgreSQL ne concerne à priori que la base de donnée, donc nous parcourons la documentation.
Et tombons sur [ce paragraphe](https://wiki.postgresql.org/wiki/Locale_data_changes#What_to_do)
>When an instance needs to be upgraded to a new glibc release, for example to upgrade the operating system, then after the upgrade
all indexes involving columns of type text, varchar, char, and citext should be reindexed before the instance is put into production.

Il existerait donc une dépendance entre la montée de version d'une librairie de l'OS (`glibc`) et le tri dans la BDD, qui pourrait expliquer ce comportement, et pour laquelle il existe une solution (la reconstruction des index).

Il n'y a à priori aucun lien entre une montée de version de la BDD et de l'OS qui la supporte, mais à bien y réfléchir, pourquoi pas ?
Scalingo met à disposition pour les conteneurs un environnement nommé [stack](https://doc.scalingo.com/platform/internals/stacks/scalingo-22-stack), qui embarque l'OS.
Le service de base de données n'utilise pas cette notion de stack, pour la bonne raison que c'est un service managé. Une montée de version de l'addon peut donc inclure des mises à jour d'OS.

Inspectons l'image docker dans ses versions successives sur le [repository Scalingo](https://hub.docker.com/r/scalingo/postgresql/tags): on voit que `glibc` a été mise à jour lors du passage de la version docker de 13.7 à 13.9.

```bash
docker run --rm -it scalingo/postgresql:13.7.0-1 ldd --version
ldd (Debian GLIBC 2.24-11+deb9u4) 2.24

docker run --rm -it scalingo/postgresql:13.9.0-1 ldd --version
ldd (Debian GLIBC 2.31-13+deb11u5) 2.31
```

Cette version est [explicitement mentionnée](https://postgresql.verite.pro/blog/2018/08/27/glibc-upgrade.html) comme embarquant des mises à jours de locales, lesquelles sont impliquées dans le tri exploité par les index, eux-mêmes utilisés pour implémenter la contrainte d'unicité. CQFD.
> For Postgres databases (..), it means that certain strings might sort differently after this upgrade.
> A critical consequence is that indexes that depend on such collations must be rebuilt immediately after the > upgrade

## S'en sortir

Si vous vous retrouvez dans la même situation que nous, à savoir que vous n'avez pas reconstruit les index, et que la base de données a été ouverte au traffic, voilà la solution que nous avons trouvée.

### Détecter les index impactés
```sql
SELECT
   indrelid::regclass::text AS table,
   indexrelid::regclass::text AS index_name,
   pg_get_indexdef(indexrelid) AS index_source,
   collname AS collation_type
FROM
   (SELECT indexrelid, indrelid, indcollation[i] coll
    FROM pg_index, generate_subscripts(indcollation, 1) g(i)
    ) s
         JOIN pg_collation c ON coll=c.oid
WHERE collprovider IN ('d', 'c') AND collname NOT IN ('C', 'POSIX');
```
Source: [Wiki PostgrSQL](https://wiki.postgresql.org/wiki/Locale_data_changes#What_indexes_are_affected)

### Rechercher les données invalides

Rechercher les doublons sur les champs protégés par les contraintes uniques utilisant les index détectés ci-dessus.
En effet, la reconstruction d'index unique n'est pas possible si les données sont en doublon.

```sql
SELECT md5(column) AS hash,
COUNT(*) FROM table
HAVING COUNT(*) > 1;
```

Déterminer quel enregistrement doit être supprimé, car créé par erreur après la montée de version.
Pour cela, s'assurer qu'aucune données liée ne soit perdue.

### Recréer les index

Il est temps de recréer les index, maintenant que les tables sont saines.

```sql
SELECT
   'REINDEX INDEX ' || indexrelid::regclass::text || ';'
FROM
   (SELECT indexrelid, indrelid, indcollation[i] coll
    FROM pg_index, generate_subscripts(indcollation, 1) g(i)
    ) s
         JOIN pg_collation c ON coll=c.oid
WHERE collprovider IN ('d', 'c') AND collname NOT IN ('C', 'POSIX');
```

Ce qui nous donne :

```sql
REINDEX INDEX users_email_unique;
REINDEX INDEX users_username_unique;
```

Assurez-vous de planifier un créneau de maintenance pour cela, la recréation d'index posant en général un verrou en écriture sur la table.
Il existe d'autres options prenant en compte les traitements concurrents, mais dans ce cas elles ne sont probablement pas indiquées.
