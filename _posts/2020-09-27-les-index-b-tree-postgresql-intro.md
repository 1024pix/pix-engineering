---
layout: post
title:  "Série sur l'index B-Tree : introduction"
date:   2020-09-27 15:30:00 +0200
categories: architecture
excerpt: Introduction à une série d'articles portant sur l'_index B-Tree_ en base de données relationnelle PostgreSQL.
cover_image: "les-index-b-tree-postgresql-intro.jpeg"
---

Nous avons choisi d'utiliser le **système de gestion de base de données relationnelle** **_PostgreSQL_**.
J'insiste et je tiens à utiliser ce terme long et pompeux, car qualifier **_PostgreSQL_** ou tout autre système de ce genre simplement de base de données est une lourde erreur.  
Un **SGBD** n'est pas seulement un coffre fourre-tout dans lequel on laisse dédaigneusement nos données. C'est un outil puissant de stockage, d'analyse et de collecte de données.

> :bowtie: L'info pour briller en société  : Postgres a pour ancêtre le projet Ingres, **IN**telligent **G**raphic **RE**lational **S**ystem

### Le "quoi" et le "comment"
Considérons une collection `individuals` dans laquelle on souhaite compter le nombre de personnes dont le prénom est Aïda. 
Dans à peu près n'importe quel langage de programmation, l'algorithme ressemblerait à cela :  
```
entier aida_compteur <- 0
Pour chaque individual dans individuals
  Si individual.firstName est "Aïda"
    aida_compteur <- aida_compteur + 1
  Fin si
Fin pour
```
En _SQL_, cela donnerait à peu près quelque chose comme ça :  
```sql
SELECT COUNT(*) FROM "individuals" WHERE "first_name" = 'Aïda';
```
Cela met en évidence la différence fondamentale entre le fait d'écrire du code dans notre travail de développeur web
de "tous les jours" et d'écrire du _SQL_.
Dans le premier cas, on doit expliciter le **_comment_**, alors que dans le second cas, on se contente simplement
d'exprimer le **_quoi_**, c'est-à-dire ce qu'on veut, mais pas comment l'obtenir.
Le SGBD parcourt-il toute la table comme nous l'avons fait pour notre code ? Garde t'il un compteur interne pour chaque prénom incrémenté à chaque modification ? Ou encore, répond-il aléatoirement :poker: ?

### Le place du dév dans les perfs en base de données
Il est acceptable pour un développeur de ne pas avoir de connaissances là-dessus pour deux raisons: 
1. Le volume de données est faible (les comportements/configurations/installations par défaut font l'affaire)
2. Vous avez un DBA (donc c'est pas votre problème)

Or chez Pix, il se trouve que :  
1. Le volume de données est important et grossit de jour en jour
2. On n'a pas de DBA mais...
3. On estime que c'est le boulot des développeurs full-stack (après tout, c'est bibi qui écrit le SQL)
4. Ca devient notre problème à la seconde où la dégradation est telle que le service n'est pas rendu honorablement (#servicepublic #exigencecitoyenne)

### Les leviers d'amélioration des perfs
Les pistes d'amélioration des performances sont multiples, les plus évidentes étant :  
- Régler plus finement les nombreux éléments de configuration
- Améliorer le hardware (RAM, processeur...)
- Améliorer les requêtes  

Le dernier point est le plus intéressant, car c'est en général là que se situent les problèmes. 
D'autant que pour les deux premiers points, vous pouvez vous heurter à des problèmes de permissions ou d'argent.  
Le dernier point en revanche est complètement à la portée du développeur. J'irai même plus loin en affirmant que c'est sa responsabilité.  

Le sujet est vaste, mais un de ses ressorts les plus efficaces est l'indexation.  
D'aucuns clament que l'indexation, du moins une mauvaise indexation, est la cause de 70% des problèmes de performance. L'indexation est un terrain idéal d'optimisation des performances pour les développeurs : 
sa mise en place n'est pas soumise a des permissions particulières, et sa conception dépend essentiellement des besoins de l'application (et c'est bibi qui code l'application !).  
Là aussi le sujet est assez vaste. Il existe plusieurs types d'index, qui répondent à plusieurs besoins. Dans cette série d'articles, nous allons nous pencher tout particulièrement sur
l'_index B-Tree_.  
L'_index B-Tree_ est utilisé dans l'amélioration des performances en lecture. En gros, dès qu'on a une requête `SELECT` qui présente du tri (`ORDER BY`, `GROUP BY`) et/ou du filtre (`WHERE`), alors on a une requête candidate à l'indexation.  
Ce sont les index les plus communs (ce sont eux derrière la commande `CREATE INDEX`), et sont adaptés lorsque :
- On cherche à sélectionner un petit sous-ensemble d'un ensemble plus grand (~10% ou moins)
- c'est-à-dire on cherche via des clauses très sélectives (beaucoup de valeurs distinctes)
 
À noter aussi que le fonctionnement d'un index B-Tree est assez similaire entre les **SGBD** les plus connus (**_MySQL_** ou **_SQL Server_** par exemple).

### Au menu

_**Partie 1 :**_ [Définition d'un _index B-Tree_, exemples et mise en place concrète]({% post_url 2020-09-28-les-index-b-tree-postgresql-partie-1 %})  
_**Partie 2 :**_ :detective_female: "Au secours ma requête est lente", ou comment identifier les candidats à l'indexation  
_**Partie 3 :**_ Index-only scan, conseils généraux et résumé des take-aways / cheat-sheet  
_**Partie 4 :**_ Présentation succincte des autres types d'index  

Je vous donne rendez-vous dans les prochains articles ! :wave:
