---
layout: post
title: "Gestion des environnements"
date: 2021-08-31 15:00:00 +0200
categories: infrastructure
excerpt: Une bonne gestion des phases d'un projet (une fonctionnalité, un changement technique, etc.) passe par une bonne gestion des environnements, depuis le poste du développeur jusqu'à la production.
cover:
    image: "gestion-des-environnements.jpeg"
    source_title: tourmag.com
    source_url: https://www.tourmag.com/COP21%C2%A0-protection-de-l-environnement-et-tourisme-sont-ils-compatibles_a77337.html
authors: jeremy_buget
---

Une bonne gestion de production ne va pas sans une bonne gestion des environnements.

Avec le temps, nous en sommes venus à considérer et gérer au quotidien 6 types d’environnements, aux besoins, contraintes et cahier des charges bien définis :
- local (ou localhost)
- review app
- intégration
- recette
- production
- autres / ad hoc

![Environnements applicatifs](/assets/images/posts/gestion-des-environnements/environnements.png)

## 1. Localhost

Il s’agit de l’environnement de développement propre à chaque développeur, sur son poste de travail professionnel individuel.

Chaque développeur est libre de travailler dans l’environnement qui lui convient le mieux (Mac OS, Linux, Windows), avec les outils qu’il préfère (WebStorm, Visual Studio, Vim).

Des outils sont développés, fournis et automatiquement mis en œuvre pour s’assurer que l’agent respecte les standards de l’organisation (versions de logiciel, format du code, etc.).

C’est le seul environnement qui n’est pas hébergé sur notre PaaS.

## 2. Review app (RA)

Il s’agit d’un environnement d’exécution (du code) éphémère, généré à chaque ouverture d’une pull request (PR) sur GitHub. Le but de ces environnements est de valider en autonomie un changement de code, qu’il soit lié à une fonctionnalité, une correction de bug, une modification technique ou une procédure / script / manipulation particulière.

Pour ce faire, nous utilisons la fonctionnalité Review Apps de Scalingo. Lorsque la PR est fermée (fusionnée ou annulée), l’environnement est automatiquement détruit.

Chaque environnement de RA reprend les spécifications techniques de la production, modulo certaines caractéristiques telles que la taille des ressources mobilisées, les données qui l’alimentent ou encore la présence / mutualisation ou non de serveurs front / CDN.

À chaque création d’une RA, des jeux de données sont générés, afin de faciliter, accélérer et fiabiliser les tests accomplis sur la version déployée.

À noter que nous avons déjà utilisé ce type d’environnement afin de tester rapidement une idée ou un prototype auprès d’un panel utilisateurs ou lors de phase de test terrain.

Autre remarque : par soucis d’économie et d’écologie, nous faisons en sorte (via Pix Bot) d’éteindre automatiquement les RA les soirs et week-end. Nous avons développé une petite bibliothèque (Node.js) Open Source pour l’occasion – scalingo-review-app-manager.

## 3. Intégration

Il s’agit d’un environnement unique, dont le but est de tester et valider la bonne intégration d’un changement sur l’ensemble du code.

Dans les faits, nous gardons un œil et une vigilance plutôt technique sur cet environnement. Mais les Product Owners (PO), de même que n’importe qui chez Pix, peuvent être susceptibles de l’utiliser pour remonter un problème, une erreur ou un feedback.

En ce sens, nous faisons en sorte que cet environnement soit disponible à tout moment.

Les données présentes au sein de cet environnement sont exclusivement des données de test. Il nous arrive quelquefois de les réinitialiser complètement ou d’en générer aléatoirement.

L’environnement d'intégration peut contenir plusieurs changements prêts à être propagés en recette.

Cet environnement est directement rattaché à la branche `dev` (sorte de branche tronc-commun) et donc automatiquement mis à jour à chaque fusion de branche / fermeture de PR.

## 4. Recette

Autre environnement unique, le but de l’environnement de recette est de contrôler plus finement les changements apportés récemment à la plateforme. Il s’agit de la dernière étape pour un changement avant la production.

Cet environnement peut aussi nous servir d’environnement de démonstration lors d’événements (notamment externes) ou de rituels (démonstration de fin d’itération), ou encore en interne pour tester les épreuves conçues par l’équipe de créateurs d’évaluations pédagogiques (postes ouverts).

Les principaux utilisateurs de la recette sont les PO qui valident les impacts fonctionnels et ergonomiques. Mais les développeurs ne sont pas en reste et contrôlent assidûment les autres points critiques de la plateforme : performance, sécurité, autres répercussions techniques ou organisationnelles.

Encore plus que l’intégration, cet environnement doit être opérationnel à chaque instant.

De même que pour l’intégration, les données sont exclusivement des données de test. Celles-ci ne sont pas réinitialisées et conservées telles quelles depuis le début, afin de se préparer et valider une dernière fois d’éventuelles manipulations ou attentions à avoir pour la production.

Contrairement à l’intégration, la recette n’est attachée à aucune branche Git. Sa mise à jour, c’est-à-dire la fusion de changements issus de la branche `dev`, est manuelle, déclenchée – la plupart du temps par les PO – via une procédure dite de “mise en recette” (ou MER).

## 5. Production

Il s’agit de l’environnement mis à la disposition des utilisateurs finaux de Pix (citoyens, partenaires, prescripteurs, certificateurs, etc.). C’est donc l’environnement le plus important de la plateforme.

En tant que tel, il se doit d’être fonctionnel, performant, sécurisé, disponible à tous, partout, tout le temps, en toutes circonstances, dans les meilleures conditions d’exploitation possibles.

Au plus fort de l’activité, l’environnement de production doit être capable d’accueillir efficacement plus d’une centaine de milliers d’utilisateurs par jour.

L’idée de cet article n’est pas de rentrer dans tous les détails opérationnels concernant la production et sa mise en œuvre. Certains éléments sont toutefois notables et intéressants pour la suite.

D’un point de vue “sécurité”, les applications sont hébergées sur des serveurs hébergés sur la zone SecNumCloud de Scalingo / Outscale. Tout est mis en œuvre pour dissocier le plus strictement possible l’environnement de production des autres environnements. Les données sont sauvegardées très régulièrement. De nombreux outils de monitoring / alerting sont développés et déployés pour assurer le parfait maintien opérationnel de cet environnement et chacun de ses composants.

Les ressources sont évidemment boostées, dupliquées et configurées en conséquence. A noter que nous utilisons pour les applications en production la fonctionnalité auto-scaling de Scalingo.

De même que la recette, la production n’est rattachée à aucune branche Git. Là aussi, le déploiement de changements en production est déclenché manuellement mais traité automatiquement via une procédure dédiée dite de “mise en production” (MEP).

## 6. Ad hoc

Il s’agit d’un dernier type d’environnement très particulier dont le but est de tester ou valider un changement, une opération, une procédure ou une idée dans un contexte particulier.

Par exemple, il nous est arrivé de monter un environnement ad hoc en zone SecNumCloud  afin de descendre certaines informations de production pour évaluer le comportement et les impacts d’un script de migration sur une table de plus de 700 millions de lignes.

Nous pouvons aussi être amenés à créer un environnement ad hoc (en zone standard) pour nous aider à configurer un outil ou un service.

Ces environnements sont toujours temporaires et détruits lorsque nous n’en avons plus l’usage.

## Remarques

Les données utilisateurs (de production) ne sont jamais descendues vers un autre environnement.

Lorsque les travaux l’obligent (ex : grosse migration de données), nous préférons créer un environnement ad hoc en zone SecNumCloud, avec des droits et des accès restreints et contrôlés.
