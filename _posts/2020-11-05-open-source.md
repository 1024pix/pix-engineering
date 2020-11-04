---
layout: post
title:  "L'open source au service du public"
date:   2020-11-03 15:07:00 +0200
categories: engineering
excerpt: TODO
cover:
  image: "open-source.jpg"
  source_title: Luca Lago, Park Güell, Barcelona, Spain
  source_url: https://unsplash.com/photos/Aa3kL3Y3NDY
authors: jeremy_buget
---

Pix a fait le choix de l'[open source](https://fr.wikipedia.org/wiki/Open_source) depuis le premier jour, avant même [le tout premier commit](https://github.com/betagouv/pix/commit/39cd1f7db03c9f40836e87976b2d6fb082a8450f) avec la toute première ligne de code. C'était il y a plus de quatre ans.

Depuis, l'équipe technique a bien grandie ; s'est réorganisée en [équipes produits]({% post_url 2020-04-13-product-teams %}) ; s'est constituée en [GIP (groupement d'intérêt public)](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000034517672/) ; a multiplié les projets, applications et outils ; a déplacé les repositories d'organisation GitHub (depuis celle de BetaGouv vers celle propre à Pix) ; a changé et fait évoluer plusieurs fois l'architecture logicielle et technique. 

Et durant tout ce temps, l'open source est resté au cœur de la politique de développement numérique de Pix et un enjeu stratégique majeur.

## Un peu d'histoire

- BetaGouv
- Éthique / convictions / valeurs personnelles
- d'abord sur le repo (archivé) de BetaGouv avant de passer sur 1024pix

## Avantages & inconvénients

Le développement de logiciel open source présente plusieurs intérêts pour un service public tel que Pix :
- respect du [principe fondamental de transparence](https://www.conseil-etat.fr/actualites/discours-et-interventions/transparence-et-efficacite-de-l-action-publique#_ftn4) de l'action publique
- vitrine technologique sur les convictions, les méthodes et le savoir-faire de Pix, ses équipes et ses collaborateurs
- obligation d'exigence et de qualité opérationnelles
- contribution à la bonne image et à l'attractivité du projet auprès de la communauté tech, pouvant découler sur des opportunités de recrutement ou d'évènementiel
- possibilité d'obtenir des contributions (remarques, informations, revues de code, remontées de problèmes, apport de changements de code, de corrections de bugs) externes spontanées
- contribue à la modernisation de l'état
- peut aider les autres projets de l'état (ex : reverse-proxy de Pass Culture, ou scalingo-review-app-manager ou matomo-deploy) 
- permet de bénéficier d'offres privilégiées

Aucun véritable inconvénient !

## Lumière sur plusieurs projets open source majeurs

- repos Pix apps et Pix sites
- ember-cli-matomo-tag-manager
- metabase-deploy
- outillage Pix Bot
- matomo-buildpack & matomo-deploy 

## Liens
- [Fil Twitter](https://twitter.com/jbuget/status/1287389472427057152?s=20)



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
