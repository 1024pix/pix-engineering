---
layout: post
title:  "Série sur l'index B-Tree : première partie"
date:   2020-09-28 09:30:00 +0200
categories: architecture
excerpt: Comment est structuré un _index B-Tree_ et quelques exemples concrets d'utilisation.
cover_image: "les-index-b-tree-postgresql-partie-1.jpeg"
---

Comprendre la structure d'un _index B-Tree_ permet de mieux comprendre ses avantages et ses inconvénients, et ainsi de l'utiliser à bon escient.

## Le concept
Comme beaucoup d'outils/stratégies/patterns utilisés dans notre métier, l'_index B-Tree_ est l'implémentation d'un concept de bon sens issu la vie réelle.  
L'exemple le plus éloquent est celui du dictionnaire. Imaginez un dictionnaire dans lequel les mots ne sont pas inscrits selon un ordre défini ! Trouver un mot dans un tel ouvrage prendrait un temps incroyable, car l'on serait obligé de tout lire jusqu'à tomber sur le mot.  
Le dictionnaire tel qu'on le connaît présente, fort intelligemment, les mots classés par ordre alphabétique. Ce n'est pas par souci esthétique, mais bien pour accélérer considérablement la recherche. Intuitivement, nous allons faire une recherche dichotomique.  
En recherchant le mot **cochléaire**, on va ouvrir un peu hasard vers le milieu le dictionnaire. Puis on va évaluer les mots sous nos yeux : "J'en suis à la lettre H, le mot que je recherche se situe donc avant.".  
En un seul mouvement, nous avons divisé l'espace de recherche par deux ! On va répéter l'opération jusqu'à trouver notre mot. La recherche est donc très efficace.  
L'_index B-Tree_ exploite très précisément ces caractéristiques.

## Un dictionnaire pas comme les autres
L'exemple ci-dessus, c'est pour vous faire comprendre le concept. Dans l'exécution, c'est un peu plus compliqué que ça. Car si c'est acceptable pour un dictionnaire de pas être mis à jour régulièrement, ça ne l'est pas du tout pour un usage en base de données.  
La structure d'un _index B-Tree_ doit donc faire face à deux contraintes majeures :  
- Pouvoir être mis à jour facilement au gré des divers `INSERT`, `UPDATE` et `DELETE`  
- Conserver, en tout temps, l'ordre défini  

C'est pourquoi la structure d'un _index B-Tree_ est en fait la combinaison d'un arbre de recherche et d'une liste doublement chaînée :
![image info](/assets/images/post_index-b-tree/index-b-tree-general.png)

### Zoom sur l'arbre de recherche équilibré
Les propriétés d'un arbre de recherche équilibré sont :
- La recherche rapide
- De garantir un temps de recherche similaire quelle que soit la valeur (la profondeur de l'arbre est identique à chaque position)

Un arbre est constitué de plusieurs nœuds. Chaque nœud non-feuille contient des éléments en nombre limité. Chaque élément est porteur de deux informations : une valeur de référence et un pointeur vers un autre nœud enfant.  
La valeur de référence est en fait la valeur la plus grande présente dans le nœud enfant pointé.  
Lorsqu'on recherche une valeur, l'arbre est parcouru de la manière suivante : on commence par le nœud racine. On parcourt chaque élément par ordre **ascendant**. Dès lors que la valeur de référence de l'élément évalué est **supérieure ou égale** à la valeur recherchée, alors on suit le pointeur de l'élément vers son noœud enfant.
On répète l'opération jusqu'à arriver au nœud feuille dans lequel se trouve la valeur recherchée.  
![image info](/assets/images/post_index-b-tree/index-b-tree-tree.png)

### Zoom sur la liste doublement chaînée
Les propriétés de la liste doublement chaînée sont :
- De pouvoir ranger de façon désordonnée les données en mémoire
- De faciliter la mise à jour et le déplacement des données  

Les nœuds feuilles de l'arbre sont organisés en liste doublement chaînée. Ainsi, si les éléments au sein d'un même nœud feuille sont rangés dans l'ordre en mémoire, les nœuds feuilles ne le sont pas.  
Ainsi, lorsqu'une nouvelle valeur est ajoutée, on évite de décaler l'ensemble des données d'un cran (ce qui revient à tout réécrire en somme). On va plutôt se positionner sur le nœud feuille adapté s'il y a de la place, à défaut en créer un nouveau.
Il suffira de mettre à jour les liens entre les différents nœuds et le tour est joué. Le contenu d'un nœud feuille diffère des autres nœuds : chacun de ses éléments, toujours classés dans l'ordre défini, est porteur de la valeur de référence bien sûr, mais aussi de l'adresse mémoire de la ligne correspondante dans la table d'origine.
Dans **_PostgreSQL_**, il s'agit du `ctid`.  
![image info](/assets/images/post_index-b-tree/index-b-tree-leaf.png)

> Idée reçue : les lignes d'une table sont rangées en mémoire dans l'ordre de la clé primaire (`id`), et ça explique pourquoi je reçois les lignes dans l'ordre de l'`id` lorsque je ne spécifie aucun ordre de tri.  
>**C'est faux!** Les lignes sont écrites en mémoire selon des règles complexes et en fonction de l'espace disponible (qui peut changer surtout après un passage de `VACUUM`). Bref, retenez que l'ordre de présentation des résultats d'un `SELECT` sans clause de tri n'est **pas** déterministe et peut changer d'une exécution à l'autre.

### Un index est une redondance utile
On comprend donc que créer un index B-Tree sur une table revient à créer une copie (incomplète, mais copie quand même) de celle-ci mais en présentant les données dans une structure spécifique destinée à la recherche rapide.  
Logiquement, chaque modification apportée à la table déclenche des opérations supplémentaires, car cette modification doit être répercutée dans tous ses index.   
En contrepartie, le parcours d'une telle structure est d'une complexité dite logarithmique : à chaque ajout d'un niveau de profondeur d'arbre, le nombre d'entrées d'index qu'on peut ajouter est multipliée par le nombre d'éléments que peut contenir un nœud.  
Pour notre arbre dont chaque nœud contient 3 éléments :

| Profondeur  | Nombre d'entrées |
| ------------- | ------------- |
| 3 | 27 |
| 4 | 81 |
| 5 | 243 |
| 6 | 729 |
| 10 | 59 049 |

> En grossissant le trait, cela veut dire qu'en 10 coups, je peux trouver n'importer quel élément dans une liste de 59 049 éléments.
 

## Du concret
Chez Pix, on propose à l'utilisateur d'évaluer son niveau de connaissances sur les compétences numériques en répondant à des questions. Chaque réponse donnée par l'utilisateur nous permet de mieux définir le contour de ses connaissances.
En fait, chaque réponse donnée va provoquer la création d'un ou plusieurs éléments de connaissance à son sujet : c'est ce qu'on appelle les `knowledge-elements`. Pour faire court, ça nous permet de retenir que l'utilisateur 123, à la date du 12 juillet 2019 à 20h03 à déclarer maîtriser cette connaissance.  
Une réponse peut provoquer la création de plusieurs éléments de connaissance. Dans la base de données, il s'agit d'une table `"knowledge-elements"`. Sans surprise, la table `"knowledge-elements"` contient une colonne `"userId"`, clé étrangère contenant l'`"id"` d'un utilisateur de la table `"users"`.  
Pour les exemples ci-dessous, on dispose d'une base de données bac à sable avec une table `"knowledge-elements"` contenant environ 1 600 000 lignes.
Cette table ne présente pour le moment aucun index sur la colonne `"userId"`. Je souhaite rechercher les `knowledge-elements` d'un utilisateur spécifique :

Note: `EXPLAIN ANALYZE` est une commande permettant d'obtenir le plan d'exécution de la requête. C'est un outil essentiel pour l'optimisation de requêtes. J'en parlerai plus en détail dans les articles suivants.
Je vais d'ailleurs retirer volontairement des morceaux du résultat de l'`EXPLAIN` et ne garder que le plus important pour cette démonstration.  
```sql
EXPLAIN ANALYZE 
SELECT * 
FROM "knowledge-elements" 
WHERE "userId" = 123456;

-> Parallel Seq Scan on "knowledge-elements"
   Filter: ("userId" = 123456)
Execution Time: 102.211 ms
```
Pour obtenir les lignes demandées, le moteur réalise donc un `Seq Scan`, un scan séquentiel. En d'autres termes, il va scanner l'intégralité de la table à la recherche des lignes demandées.  
Cela lui a pris 102.211 ms, sachant qu'il a manifestement sollicité plusieurs workers pour réaliser cette opération (`Parallel`).  
Il se trouve que 102 ms, c'est un peu long (si si), et qu'en plus c'est une requête qu'on est amené à exécuter très souvent. Chez Pix, elle fait même partie du top 3 des requêtes les plus souvent exécutées (presque 180 000 fois par jour).  
Essayons de positionner un index sur la colonne `"userId"`.  
```sql
CREATE INDEX knowledge_elements_userid_idx ON "knowledge-elements" ("userId");

EXPLAIN ANALYZE 
SELECT * 
FROM "knowledge-elements" 
WHERE "userId" = 123456;

->  Index Scan using knowledge_elements_userid_index on "knowledge-elements"
    Index Cond: ("userId" = 123456)
Execution Time: 0.151 ms
```
Ce coup-ci, le moteur effectue un `Index Scan` en utilisant l'index créé auparavant. La requête s'est exécutée près de **680** fois plus vite. 

### Ne pas succomber à la tentation
Malgré l'argument de vente exceptionnel présenté ci-dessus, je vous enjoins à ne pas tomber dans le piège de la création d'index en pagaille.
Je vais vous donner deux exemples classiques des cas où cela ne sert à rien du tout.  

##### Une valeur trop commune
Reprenons les éléments de connaissance dont on a parlé tout à l'heure. Une autre information associée à l'élément de connaissance est si l'élément de connaissance a été validé ou non par l'utilisateur.
C'est représenté par une colonne `"status"` dans la table. Le nombre de valeurs distinctes que peut avoir cette colonne est très limité (`'VALIDATED'`, `'INVALIDATED'` et `'RESET'`). Essayons de récupérer l'ensemble 
des éléments de connaissances validés par nos utilisateurs.  
```sql
EXPLAIN ANALYZE 
SELECT * 
FROM "knowledge-elements" 
WHERE "status" = 'VALIDATED';

-> Seq Scan on "knowledge-elements"
  Filter: ((status)::text = 'VALIDATED'::text)
Execution Time: 401.903 ms
```
Avec un index, ça ira certainement plus vite, non ?  
```sql
CREATE INDEX knowledge_elements_status_idx ON "knowledge-elements" ("status");

EXPLAIN ANALYZE 
SELECT * 
FROM "knowledge-elements" 
WHERE "status" = 'VALIDATED';

-> Seq Scan on "knowledge-elements"
  Filter: ((status)::text = 'VALIDATED'::text)
Execution Time: 411.637 ms
```
L'index n'est même pas utilisé par le moteur.  
Si la recherche dans un index pour une valeur spécifique est rapide, l'opération de récupération du ctid dans l'entrée de l'index, puis de récupération de la ligne correspondante dans la table est assez coûteuse.
Lorsqu'il s'agit de faire le pont sur un petit ensemble de valeurs entre l'index et la table, ce coût est négligeable au regard du gain apporté par l'efficacité de la recherche.  
Par contre, s'il faut le faire pour un grand ensemble de valeurs, alors cela ne vaut plus le coup/coût. Dans notre cas, le moteur a estimé que
la clause `WHERE "status" = 'VALIDATED'` n'était pas suffisamment restrictive. En d'autres termes, le moteur, à l'aide de ses statistiques, sait que beaucoup de lignes
(en l'occurrence près de 60%) ont pour valeur `'VALIDATED'` dans leur colonne `"status"`. Il a donc estimé que cela reste moins coûteux de parcourir toute la table que de devoir utiliser l'index.
 
##### Une table trop petite
On dispose d'une table `"pix-roles"` qui contient la liste des rôles spécifiques qu'un utilisateur peut avoir dans certains cas. Cette table contient très peu d'entrées (moins de 10).  
Je souhaite obtenir la ligne de la table répondant au rôle `'Magicien'`:
```sql
EXPLAIN ANALYZE 
SELECT * 
FROM "pix-roles" 
WHERE "name" = 'Magicien';

-> Seq Scan on "pix-roles"
  Filter: ((status)::text = 'Magicien'::text)
Execution Time: 0.022 ms
```
En ajoutant l'index.
```sql
CREATE INDEX pix_roles_name_idx ON "pix-roles" ("name");

EXPLAIN ANALYZE 
SELECT * 
FROM "pix-roles" 
WHERE "name" = 'Magicien';

-> Seq Scan on "pix-roles"
  Filter: ((status)::text = 'Magicien'::text)
Execution Time: 0.019 ms
```
L'index n'a pas été utilisé car le moteur a estimé qu'il était moins coûteux de parcourir la table entièrement plutôt que de s'embêter à utiliser l'index. 

##### Les questions à se poser avant d'ajouter un index
- L'index existe t'il ? (et n'est pas utilisé pour des raisons abordées dans le prochain article, #teaser)
- La table est-elle suffisamment grande ?
- La colonne indexée présente t'elle suffisamment de valeurs distinctes ?
- L'index et sa mise à jour va t'il affecter grandement les performances en insertion / mise à jour / suppression (oubliez l'index si votre table est très peu lue, mais très souvent modifiée)
- L'index va t'il remédier aux requêtes actuellement lentes (_YAGNI_, concentrez-vous sur les requêtes qui posent problèmes _aujourd'hui_)
- Les expérimentations d'index sont-elles réalisées sur un environnement proche de la production ?

#####  Ajouter un index sur une base en production
Je lève une petite alerte ici. La création d'un index **bloque** la table ciblée en **écriture**. Et pour une table un peu conséquente, la création d'un index peut prendre du temps.  
C'est évidemment un risque à ne pas prendre sur un environnement de production. Heureusement pour nous, PostgreSQL propose une création d'index _production-friendly_ (plus généralement, PostgreSQL propose beaucoup de commandes _production-friendly_ :hugs:).  
On peut ajouter l'option `CONCURRENTLY` à la commande de création :
```sql
CREATE INDEX CONCURRENTLY knowledge_elements_status_idx ON "knowledge-elements" ("status");
```
La contrepartie est évidemment un allongement du temps d'exécution de la commande. Je vous invite à consulter la documentation officielle [pour plus d'infos](https://www.postgresql.org/docs/13/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY).

Le prochain article vous montrera comment partir à la traque d'une requête lente et comment y remédier à l'aide d'un _index B-Tree_.
