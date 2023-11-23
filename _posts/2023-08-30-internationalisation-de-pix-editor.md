---
layout: post
title:  "Internationalisation de Pix Editor"
date:   2023-08-30 17:02:00 +0200
categories: internationalisation
excerpt: Découvrez le travail d'internationalisation effectué pour Pix Editor
cover:
    image: "internationalisation.jpg"
    source_title: World Map
    source_url: https://images.unsplash.com/photo-1562504208-03d85cc8c23e?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=3540&q=80
authors: iris_benoit
---

Pix s'internationalise et doit se rendre facilement accessible pour d'autres pays. Qui dit d'autres pays, dit d'autres langues. Ici, je vais couvrir le travail qui est fait pour internationaliser sereinement Pix Editor, puisque c'est l'un des deux repos sur lequel je travaille au quotidien.
Cela fait bientôt 5 mois (!!) que je suis chez Pix, et je découvre encore beaucoup de choses de ce gros projet qu'est Pix ! Cet article aura pour but de montrer les différentes étapes (qui sont encore en cours) vers l'internationalisation. C'est donc de la réflexion sur le fil de nos pensées respectives et des problèmes sur lesquels on bute, immanquablement.

## 1. Le besoin

Le besoin est simple : Pix s'étend dans toute l'Europe, et bien qu'il existe déjà une version anglaise de nos épreuves, et quelques éléments spécifiques de Pix en espagnol, il était temps de rendre le tout facilement internationalisable. C'est-à-dire pouvoir ajouter facilement une nouvelle langue, puis traduire (via contribution manuelle ou via une API dédiée) tous les éléments de notre site.
La problématique est un peu plus complexe. Tout d'abord, il faut savoir que chez Pix, nous utilisons à la fois Airtable et une base de données PostgreSQL.

Pourquoi Airtable ? Il faut revenir aux origines de Pix, à l'époque où tout était léger et simple. Airtable semblait la solution idéale. Avec le temps et l'agrandissement du projet, il est devenu compliqué de s'en passer tout en devenant compliqué d'y rester.
Je dois admettre que la solution trouvée pour pallier aux limitations d'Airtable (qui est long et ne fait des requêtes que par lots de 10) est assez élégante. Pour simplifier : toutes les 24h, on va sauvegarder les données d'airtable dans une _release_ qui sera mise en cache via _Redis_ pour que les données restent rapidement accessibles.

Néanmoins, avec le temps, la solution Airtable devient de plus en plus compliquée à tenir, et la transition vers une base de données Postgre est la solution naturelle qui s'impose. Pour autant, le temps de faire une transition propre et sans bavures, on doit faire avec le fait de bosser sur de la base de données hybride : en partie sur Airtable et en partie du Postgre.

## 2. La conception

> pour simplifier l'écriture (et la lecture) de cet article, **base de données** sera noté **BDD** et **postgre** deviendra **PG**.

Pour en revenir à l'internationalisation, en toute logique, nous choisissons de créer une nouvelle table PG nommée _translations_
Elle contiendra un index (qui sera défini comme une clé composite des 2 champs suivants), une _key_, une _locale_ et une _value_

La key fera référence à l'élément appelé que l'on veut traduire (un titre/une description d'un compétence, une épreuve, une thématique, etc.), la locale indiquera la lanque et la valeur sera donc la traduction correspondante.

> Il est important de noter la différence entre **langue** et **locale** qui n'indiquent pas la même chose. Certaines de nos épreuves sont en locale fr-fr, car elles font références à des éléments franco-français, comme une adresse postale ou un service public français.

Le travail qui se fait lors de la conception consiste en des discussions, parfois des schémas (souvent que j'effectue pour être sûre d'avoir bien compris la problématique et la solution proposée) et une plongée dans l'inconnu (ou presque) !

Au bout de plusieurs tickets effectués, on réalise qu'il va falloir tout de même travailler avec Airtable, notamment parce que notre **environnement de dev'** et ce qu'on appelle la **Review App** (ce qui est déployé lors de la création d'une Pull Request, une sorte de répétition générale avant la mise en prod') déploient avec une base de données vide.
