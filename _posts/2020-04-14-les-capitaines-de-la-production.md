---
layout: post
title:  "Les Capitaines de la Production"
date:   2020-04-14 15:07:00 +0200
categories: organisation
excerpt: Comment nous avons constitué une équipe spéciale de développement dédiée au suivi de production.    
cover_image: "les-capitaines-de-la-production.jpg"
---

Avec l’évolution du nombre d’utilisateurs / partenaires / clients / collaborateurs / équipes / outils / process / etc. il devient vital pour Pix de consacrer des forces au suivi de production (personnes, ressources matérielles et immatérielles) et de bien les organiser.

**Les capitaines** (de la production) désigne le dispositif en réponse à ce besoin.

**L’équipage** est composé de développeuses ou développeurs dédiés (“les capitaines”) pour un temps aux problématiques de gestion de la production.

## Missions

- **Monitorer** : mettre en place des dashboards et systèmes pour surveiller la prod en temps réel et pouvoir anticiper les besoins futurs.
- **Réparer** : lorsqu’un incident se produit, être en mesure de relancer la prod rapidement.
- **Entretenir** : faire évoluer la prod, renforcer ses points faibles pour éviter les incidents.
- **Former** : capitaliser l’expérience de la team Capitaines de la Prod pour permettre au reste de l'équipe de monter en compétence sur les sujets notamment SRE. 
- **Informer** : Communiquer au reste de Pix de manière à prendre les mesures pour gérer les crises et avertir les utilisateurs dans les meilleures conditions.

## Objectifs

- Alerting : 
  - être alerté d’un problème de production < 1 mn
- Gestion de crise ou d’incidents : 
  - remettre le système en état de fonctionner < 10 mn
  - corriger le problème de façon pérenne < 10 jours ouvrés
  - s’organiser, communiquer et réagir de sorte à générer < 10 tickets Freshdesk
- Formation : 
  - 2/3 devs se sentent à l'aise (>= 3/5) à prendre et assumer le rôle de capitaine

## Organisation

### Droits & devoirs

Les capitaines sont libérés de toutes contraintes liées à leur équipe produit respective afin d’accomplir les missions et chantiers de leur équipage.

Par ailleurs, la Production étant prioritaire, l’équipage peut réquisitionner des devs ou tout(e) autre profil / expertise si besoin.

### Fonctionnement en  “quart biseauté”

La durée d’un capitanat est de 2 semaines. Chaque équipage est formé pour une durée d’une semaine. 

À l’issue d’une semaine, l'équipage opère un roulement : le capitaine le plus ancien transmet son rôle à un développeur qui devient à son tour capitaine, et réintègre son équipe produit ; l’autre capitaine d'équipage reste en place pour assurer la continuité des chantiers. Afin de partager les connaissances de supervision à tous les dev Pix, chaque semaine un captain d’une équipe métier différente sera désignée .

## Journal de bord

À la fin de chaque semaine, l'équipage produit une communication (page de journal, entrée de wiki, mini compte-rendu sur Slack ou par e-mail) à destination des équipes produit et du prochain équipage.

Dans l’idéal, le compte-rendu couvre les problèmes rencontrés et leur résolution ainsi que les chantiers en cours, permettant au nouvel équipage de prendre le relai. 

On annonce à ce moment le nouveau Capitaine.

> Pas besoin. d’en faire des caisses. 3 paragraphes peuvent suffire.

## Outils

Les outils des capitaines sont décrits sur cette page.

Les capitaines de la production peuvent échanger sur la chaîne Slack #team-captains.

Les chantiers planifiés peuvent être suivis via un Trello.

Un guide pour débugguer la prod est disponible : Que faire en cas de problème en production ? 

