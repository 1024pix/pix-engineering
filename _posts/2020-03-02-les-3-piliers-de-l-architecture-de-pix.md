---
layout: post
title:  "Les 3 piliers de l'architecture de Pix"
date:   2020-03-02 00:00:01 +0200
categories: architecture
excerpt_separator: <!--more-->
---

L'architecture de Pix est construite autour de 3 propriétés applicatives et leurs contraintes respectives : 

- User centric architecture, 
- Platform oriented architecture
- API-based architecture

<!--more-->

## 1. User Centric Architecture

D'un point de vue architecture Pix a été conçu comme une galaxie avec des planètes, une par usage / typologie d'utilisateurs :

- [Pix.fr](https://pix.fr) pour les utilisateurs invités (i.e. non connectés) ou en quête d'information institutionnelle
- [Pix App](https://app.pix.fr) pour les utilisateurs connectés qui souhaitent se positionner au sein du référentiel de compétences numériques et se faire certifier
- [Pix Orga](https://orga.pix.fr) pour les organisations (établissements scolaires ou supérieurs, entreprises, administrations, associations, etc.) qui déploient, coordonnent et/ou prescrivent Pix auprès de populations d'utilisateurs accompagnées (écoliers, étudiants, salariés, etc.)
- [Pix Certif](https://certif.pix.fr) pour les référents de centres de certification qui organisent, font passer et transmettent les syntèhses des sessions de certification
- [Pix Admin](https://admin.pix.fr) pour les administrateurs de la plateforme : développeurs, jurés, exploitants, etc.
- [Pix Editor](https://editor.pix.fr) pour les producteurs / réplicateurs de contenu pédagogique (domaines, compétences, épreuves, acquis, tutoriels) qui souhaitent enrichir le référentiel Pix
- etc.

Le choix d'une plateforme Web (VS. client lourd "à l'ancienne" VS. RDA VS. mobile / native / hybride) a été fait afin de toucher un public le plus large.

Il est important que n'importe qui puisse utiliser le bon espace de la plateforme dans les bonnes conditions.

Ex : pouvoir passer des épreuves mêmes dans les transports, mais tout en étant prévenu d'une expérience dégradée.

Ex : disposer d'une technologie permettant d'assurer un maximum l'accessibilité au plus grand nombre (d'un point de vue technique à défaut de celui fonctionnel ou pédagogique).

Ex : être en capacité de scaler rapidement, efficacement et en toute transparence et sécurité pour tous les (types de) utilisateurs.

Ex : être en capacité de tester rapidement des protoypes (en mode Lean Startup) au plus proche du terrain.

Ex : faire en sorte que chaque brique logicielle ait un minimum d'impact sur le système au global en cas de défaillance, afin d'assurer une qualité et une continuité de service optimales.

Les caractéristiques d'une architecture orientée utilisateur sont :

- utili(sabili)té
- accessibilité
- disponibilité

### Utili-sabili-té

Chaque fonctionnalité de chaque application doit être le fruit d'un besoin réel, vérifié et validé auprès, avec et sur le terrain. Si une fonctionnalité ne valide pas ou plus les hypothèses métier émises, alors elle doit évoluer ou être retirée.

De même qu'elles doivent être utiles, chaque fonctionnalité doit être utilisable, c'est-à-dire se révéler pratique - ergonomique - pour les utilisateurs concernés et leur apporter immédiatement une valeur ajoutée sans contrepartie.

Ex : limiter le besoin de transfert (téléchargement & téléversement) de fichiers de différents formats au profit d'une interface, de tableaux et d'interactions accessibles en moins de 3 clics.

### Accessibilité

Une architecture user-centric doit être exploitable par la totalité de ses utilisateurs.

Dans la mesure où Pix est un service public, la plateforme doit être exploitable par l'ensemble des citoyens / personnes souhaitant ou devant l'utiliser : les personnes en situation de handicap, les personnes valides, les personnes disposant de matériel ancien ou de faible puissance, les personnes disposant d'infrastructure instables ou dépassées (ex : 3G, localisations sans accès Internet, transports), etc.

Ex : il doit être possible de naviguer au clavier

Ex : chaque page doit nécessiter moins de 3Mo de données à récupérer

Ex : chaque interaction doit produire un résultat en moins de 3s

### Disponibilité

L'ensemble des applications de la plateforme doit être disponible à tout moment, à tout endroit, en toute condition.

## 2. Platform Oriented Architecture

De même que chaque application mobile qui se développe tend à devenir une messagerie sociale, tout système d'information qui se développe tend à devenir une plateforme numérique.

Pix a été pensé depuis le premier jour comme un "SI plateforme". 

Le but d'une plateforme est de permettre à ses utilisateurs d'accomplir un maximum d'actions au sein d'un unique écosystème, avec un minimum de changement de contexte ou d'interactions extérieures, pour un domaine donné – lequel tend à s'étendre et se diversifier avec le temps et le succès.

Les caractéristiques d'une plateforme sont :

- l'homogénéité
- l'extensibilité
- l'interopérabilité

### Homogénéité

Le tout doit paraître cohérent, consistant, fluide, similaire, avec une philosophie simple, claire et persistante.

À travers les différents espaces qui lui sont proposés, l'utilisateur doit comprendre qu'il navigue dans des endroits différents pour l'usage, mais semblables dans leur approche et leurs mécanismes.

Ex : toutes les applications front sont développées avec Ember.js afin que tout développeur puisse intervenir dans les meilleures conditions / délais sur n'importe quelle application avec une qualité (supérieure) identique sur l'ensemble des applications.

Ex : toutes les sources de données sont accédées via des objets Repository et Data Source, qui sont des abstractions en vue d'isoler le domaine métier tout en proposant une approche similaire.

Ex : un maximum d'applications sont déployées de la même façon (git push), chez le même hébergeur (Scalingo) afin de faciliter et d'accélérer la mise en place et l'exploitation d'une application nouvelle ou existante.

### Extensibilité

Une plateforme qui ne se développe pas finit par imploser ou dépérir. Pour bien se développer une plateforme doit pouvoir être étendue d'un point de vue technique, fonctionnel mais aussi organisationnel.

Ex : découper les usages en plusieurs applications, qui peuvent être développées et déployées en parallèle.

Ex : s'appuyer sur des logiciels / cadriciels / outils Open Source et/ou maîtrisés par le plus grand nombre.

Ex : coller au plus près des concepts, procédures et protocoles standards. 

Ex : favoriser l'ingénierie logicielle de qualité (tests, automatisation, outils d'exploitation, industrialisation, rationalisation des ressources, etc.).

Remarque : dans cette optique, il n'est pas exclu un jour de passer à une architecture applicative orientée micro-services, voire serverless type AWS Lambda

### Interopérabilité

Dans un monde de plus en plus connecté, il est vital pour une plateforme de pouvoir s'interfacer, échanger et communiquer des données / process / informations avec tout un tas (pour ne pas dire "un maximum") d'acteurs : partenaires ou non, sur le même domaine ou pas.

La meilleure façon d'y parvenir est de :

- respecter et implémenter les standards (en priorité)
- proposer un maximum de connecteurs (techniques, fonctionnel) dans un maximum de formats (HTTP, XML, JSON, binaire, etc.)

Ex : fournir un mécanisme de SSO entre les différentes applications front (échanges internes).

Ex : fournir une API sécurisée opérationnelle et pertinente (échanges externes).

## 3. Restful API-based Architecture

On considère 4 types de données : 

- les données à caractère juridique ou d'archivage (logs, fichiers à valeur de preuve, scans, etc.)
- les données référentielles, de type contenu pédagogique 
- les données générées par les utilisateurs, leurs interactions et leurs usages (comptes, évaluations, réponses, snapshots, organisation, campagnes, etc.)
- les données éditoriales ou marketing

Quelque soit le type de données, elles ne doivent être accédées ou remontées aux utilisateurs / applications front qu'au travers d'API(s) métier.

L'API Pix respecte les caractéristiques suivantes :

- style d'architecture : REST
- protocole d'échange : HTTPS
- format d'échange : [JSON API](http://jsonapi.org/)

Le format JSON API a été adopté car il est celui développé et préconisé par Ember Data.
