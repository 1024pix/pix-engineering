---
layout: post
title:  "Point Tech, kezako ?"
date:   2021-10-29 17:06:20 +0200
categories: organisation, management
excerpt: "Le point tech est un événement bi-mensuel interne au GIP Pix, au sein du pôle engineering (40 personnes), qui a lieu un mercredi après-midi sur deux, au cours duquel chacun peut être amené à travailler seul ou en groupe sur le sujet de son choix."
cover:
  image: "point-tech.jpg"
  source_title: David Siglin (unsplash.com)
  source_url: https://unsplash.com/photos/szVTIkisN1M
authors: jeremy_buget
---

## TL&DR;

Le point tech est un événement bi-mensuel interne au GIP Pix, au sein du pôle engineering (40 personnes), qui a lieu un mercredi après-midi sur deux, au cours duquel chacun peut être amené à travailler seul ou en groupe sur le sujet de son choix. À la fin du point, chaque sujet est présenté et partagé aux autres participants ou spectateurs curieux.

## Introduction

Au même titre que le Daily Meeting ou que la Démonstration de fin de Sprint, le Point Tech fait partie des événements ritualisés les plus anciens qui structurent la vie du pôle Engineering de Pix.

Il est aussi l’un de ceux ayant le moins évolué au cours du temps, malgré les nombreux mouvements opérés (arrivées et départs de personnes, création, split ou réorganisation des équipes).

Concrètement, il s’agit d’une après-midi entière, inter-équipe, qui se tient toutes les deux semaines, le mercredi après-midi, en alternance avec la démonstration du sprint, au cours de laquelle les développeuses et développeurs qui le souhaitent, peuvent travailler librement sur un ou plusieurs sujets de leur choix (technique, fonctionnel, organisationnel, documentaire, etc.), seul ou en groupe, en vue d’améliorer le quotidien de l’équipe Engineering, en termes de plateforme, de code, d’outils et process, de standards ou d’interactions (mécaniques ou humaines).

L'événement est non-obligatoire. Dans les faits, on constate une participation active régulière de l’ensemble des personnes et des équipes, ce qui contribue à la qualité, la richesse et le plaisir du moment.

## Sujets

Les sujets abordés sont extrêmement variés :
- remaniement d’un bout de code jugé problématique
- kata de code, sur des technos utilisées sur le projet, ou pas
- mise en œuvre et exploration d’une technologie sur le projet (Cypress, Testing Library, BullMQ, etc.)
- discussion ouverte
- intégration et configuration d’outils de productivité ou de qualité du code
- montée de version de dépendances ou frameworks
- optimisation de la taille des ressources JS ou CSS
- revue et audit de sécurité applicative
- nettoyage et remise en forme de notre espace documentaire
- analyse de cas ou de comportements système suspects
- etc.

Certains sujets abordés ne peuvent être couverts et traités complètement en une après-midi. Le cas échéant, ils peuvent être poursuivis lors des points tech suivants. C’était le cas, par exemple, de la mise en œuvre de tests end-to-end (E2E) avec Cypress ; du remplacement des bases de données SQLite sur les environnements de développement par des bases PostgreSQL, pour être au plus proche de la production (merci Docker 🐳) ; ou bien encore, du développement d’une [action GitHub](https://github.com/1024pix/pix/blob/dev/.github/workflows/auto-merge.yml) pour fusionner automatiquement les branches et pull requests (PR) validées et prêtes pour la production.

D’autres sujets peuvent tout aussi bien être abandonnés ou délaissés quelque temps. C’est déjà arrivé et ça arrivera encore. Dans ce cas, nous avons une règle stipulant que les PR de plus de 30 jours d’inactivité doivent être fermées non-fusionnées.

## Déroulement

Le point tech a lieu un mercredi après-midi sur deux, de 13h45 jusqu’à 17h45 environ.

- **[13h45]** : Un rappel automatique annonce le début du point tech. Les participants sont invités à rejoindre la visioconférence et le tableau interactif virtuel.
- **[13h45-14h00]** : Chacun indique sous forme de post-it virtuel le sujet sur lequel il compte travailler
- **[14h00-14h15]** : Chaque post-it est évoqué, de préférence par son auteur, pour expliquer ce qu’il recouvre
- **[14h15]** : Top départ ! Les personnes ou groupes se lancent sur leurs chantiers respectifs
- **[14h15-17h00]** : Les chantiers avancent. Personne n’est bloqué sur un sujet ou une équipe. Ceux qui le souhaitent peuvent changer de groupe ou de chantier à tout moment (lois des 2 pieds, issue du [forum ouvert](https://fr.wikipedia.org/wiki/M%C3%A9thodologie_Forum_Ouvert).
- **[16h45]** : Un rappel automatique prévient tout le monde de se préparer pour la restitution.
- **[17h00-17h45]** : C’est l’heure du “Spectacle” ! Un·e animateur·rice invite à tour de rôle les groupes à restituer le fruit de leurs efforts. C’est le moment où l’on découvre et partage un maximum de choses, des apprentissages, des décisions, etc. La réunion est ouverte à tous, et des gens n’ayant pas participé à l’après-midi peuvent assister au résultat en tant que spectateur. Il faut compter entre 2 et 5 minutes par sujet (7mn max) afin que tout le monde puisse passer correctement. Au pire, les personnes et leurs sujets sont identifiés. Les discussions pourront se poursuivre plus tard avec ceux qui le souhaitent.
- **[17h45]** : Fin du point tech. L’occasion de s’intéresser, poursuivre ou creuser des sujets autour d’une bonne bière ou d’une bonne bolée de cidre (ou tout autre liquide de bon goût).

## Histoire

> “Un bon artisan ne néglige ni ses outils, ni sa santé.”

Faire les bonnes choses, de la bonne façon, dans les bonnes conditions requiert de l’investissement de temps, de compétences, d’énergie. Partant de ce principe, nous avons très tôt ressenti le besoin de se réserver régulièrement du temps pour améliorer nos outils, notre chaîne de production, nos pratiques et nos standards de code.

Initialement, nous prévoyions 1h tous les mardis en fin de matinée (car nous comptions sur la contrainte naturelle du midi et de nos estomacs pour nous timeboxer).

C’était bien pas insuffisant. À mesure que le projet et l’équipe grandissaient, nous étions frustrés de ne pouvoir traiter plus en profondeur, plus de sujets.

C’est ainsi qu’il y a 3 ans (2017), nous sommes passés de 1h toutes les semaines à 4h toutes les 2 semaines. Traiter des sujets compliqués, voire complexes, devenait possible.

Autre évolution notable : nous sommes passés d’un traitement séquentiel et collectif par sujet unique à une organisation parallèle multi-sujets / multi-groupes / cross-équipes. À partir de cet instant, tout le monde ne pouvait plus suivre et s’investir dans tous les sujets de l’après-midi, mais plus de sujets pouvaient être abordés au global de l’organisation. Cela nécessite une répartition intelligente a minima des sujets, pour que des personnes nouvelles ou juniores, au sein de Pix ou sur le sujet en question, ne soient pas livrées à elles-mêmes. Ainsi qu’un niveau de confiance réel et une bonne communication entre les personnes.

Cette pratique et cette organisation représentent un réel investissement du point de vue du GIP Pix, mais les résultats ont toujours été à la hauteur (cf. section “Bilan” ci-dessous). Ainsi, la pratique n’a jamais été vraiment remise en question.

## Réussites

Depuis la création du point tech, de très nombreuses réalisations marquantes ont vu le jour et ont été déployées en production, pour le plus grand bien des utilisateurs comme des agents du GIP Pix.

Nous avons cité précédemment la mise en place de Cypress, la généralisation de PostgreSQL à tous les environnements et le développement d’une action GitHub d’auto-merge.

On peut ajouter à ces réussites les exemples / projets / initiatives ci-dessous :
- le développement d’une bibliothèque de composants graphiques Ember – [Pix UI](https://github.com/1024pix/pix-ui/tags) – pour rationaliser et mutualiser les éléments graphiques entre les applications front
- la mise en place du [Design System Pix](https://storybook.pix.fr/?path=/story/utiliser-pix-ui-installation--page) (via StoryBook) pour mettre en avant et documenter cette bibliothèque de composants
- le développement et la publication d’un addon Ember pour le support de Testing Library : [@1024pix/ember-testing-library](https://www.npmjs.com/package/@1024pix/ember-testing-library)
- le développement et la publication d’un addon Ember pour le support de Matomo tag Manager : [@1024pix/ember-cli-matomo-tag-manager](https://www.npmjs.com/package/ember-cli-matomo-tag-manager)
- la création et l’enrichissement au fil du temps d’un bot - [Pix Bot](https://github.com/1024pix/pix-bot) - pour nous aider dans notre quotidien de développeurs et développeuses, en simplifiant, fluidifiant et sécurisant l’ensemble des actions critiques que nous sommes amenés à opérer, notamment la mise en production d’une nouvelle version de la plateforme
- la configuration, l’amélioration et le suivi régulier de dashboards et autres outils de monitoring techniques
- le développement ou la mise en place de tout un tas de notifications et autres connecteurs entre nos différents outils du quotidien
- différents audits ou réflexions, mené(e)s entièrement ou initié(e)s avant de donner lieu à des projets plus conséquents, sur des aspects critiques tels que la sécurité, l’accessibilité, les performances, la gestion des droits (RGPD), ou encore l’internationalisation

Tous ces sujets ne représentent qu’une infime partie de tous les chantiers gérés lors des points tech. Les occasions de se réjouir ne manquent pas ! Et chacune représente une avancée concrète pour la plateforme, la productivité et les conditions de travail de chacun.

Quel plaisir de déployer le mécanisme d’auto-merge et de n’avoir plus à se soucier de suivre ses PR et les rebaser régulièrement ! Quel plaisir encore de voir le build d’intégration continue retrouve une stabilité fiable et confortable, ou prenne sensiblement moins de temps ! Quel plaisir toujours de voir que les dépendances, leur version ainsi que leur usage au travers des différentes applications de la plateforme ont été ré-alignés !

## Bilan

Au vu des très nombreuses réussites évoquées ci-dessus, la pratique organisationnelle des points tech telle que nous la mettons en œuvre chez Pix est incontestablement un succès.

Mais au-delà de l’aspect opérationnel, le point tech permet de brasser régulièrement les équipes et de permettre à des personnes qui ne se côtoient pas si souvent de rencontrer et travailler avec de nouvelles personnes. Sans aller jusqu’à parler de team building, c’est un bon moyen pour tisser des liens et faire en sorte que chacun se comprenne bien et interagisse de la meilleure façon avec l’autre, qui sera peut-être son futur partenaire d’équipe. Connaître l’autre, c’est aussi se rendre compte qu’il s’intéresse ou maîtrise une compétence qui peut nous intéresser nous-mêmes ou être utile un jour. Enfin, c’est une façon de partager des points de vue ou de découvrir de nouvelles façons de faire.

Le point tech est aussi l’occasion pour les développeurs et les développeuses, internes ou prestataires, de se former et d’approfondir leurs connaissances / compétences. Réaliser des productions de qualité nécessite d’avoir des bases solides quant à ses outils ou son domaine d’activité. Et pour cela, il faut pratiquer et répéter régulièrement son artisanat.

Il arrive parfois que certaines équipes connaissent des moments de tensions critiques ou à fort enjeu. Les points tech sont l’occasion pour celles et ceux qui en ont besoin de sortir la tête du guidon, souffler, ou trouver de l’aide.

Pour finir, c’est une pratique qui permet de conserver une dynamique positive au sein de l’équipe Engineering, avec des personnes motivées, intéressées, impliquées, curieuses, ouvertes et épanouies.

## Conclusion

Le point tech est une pratique organisationnelle parmi d’autres. Pour Pix, c’est un événement qui rythme le calendrier et les itérations des équipes techniques de façon très positive, depuis longtemps. Il a fallu l’adapter à mesurer que le service évoluait, mais aujourd’hui encore, c’est un rituel inter-équipes commun qui porte ses fruits, aussi bien sur le plan individuel que collectif.

Si vous avez des questions, [contactez-nous](https://github.com/1024pix/pix-engineering/issues) pour que nous puissions améliorer cet article !
