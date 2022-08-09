---
layout: post
title:  "Principes de sécurité"
date:   2020-04-13 19:33:00 +0200
categories: architecture securité
excerpt: Principes de sécurité technique que nous nous appliquons et qui guident nos choix de design applicatif.
cover:
image: "principes-de-securite.jpg"
source_title: zhuanlan.zhihu.com
source_url: https://zhuanlan.zhihu.com/p/140799707
authors: jeremy_buget
---
_Cet article date d'avril 2020, et a été écrit par l'ancien CTO de Pix Jérémy Buget. Il représente une vision de la sécurité au sein de Pix dans un contexte différent de celui actuel. L'application de Pix a pu évoluer depuis cet article._

D'un point de vue technique et pour le développement des applications, Pix repose sur les 3 principes directeurs suivants :

- _Humility Driven Policy_
- _Less (personal data) is more (people tranquility)_
- _The closer to the data, the more safety features_


## 1. Humility Driven Policy


En matière de protection d'un SI, l'humilité est la première des sécurités.

Se dire qu'on maîtrise et se reposer sur son talent, aussi grand soit-il, c'est prendre le risque de tomber sur quelqu'un (personne ou organisation concurrente) de malintentionné disposant de plus de compétences, temps, ressources, volonté que nous.

La plupart des failles de sécurité :
- résultent de l'exploitation d'une addition (voire d'une multiplication) de failles
- dont l'une au moins inclut une négligence ou une erreur humaine

Une politique de sécurité basée sur l'humilité se décline sur les préceptes suivants :

- des standards plutôt que des idées nouvelles
- des experts habitués plutôt que des talents curieux
- des moyens adéquats plutôt que les contraintes du moment

### 1.1 - Des standards plutôt que des idées nouvelles

Plutôt que de réinventer la roue avec l'illusion d'avoir la nouvelle idée ou la nouvelle compétence que personne n'aurait eu avant nous, il est préférable d'apprendre et de mettre en oeuvre des solutions déjà connues, reconnues et éprouvées avec succès par des milliers d'individus et d'entreprises sur autant de projets à travers le monde et les années.

Ex : les services de l'API implémentent le [protocole AAA](https://fr.wikipedia.org/wiki/Protocole_AAA) : Authentification, Autorisation, Traçabilité.

Ex : l'authentification au sein de la plateforme se base sur le [protocole OAuth 2](https://oauth.net/2/), cf. [RFC 6749](https://tools.ietf.org/html/rfc6749), [RFC 6750](https://tools.ietf.org/html/rfc6750) et [RFC 6819](https://tools.ietf.org/html/rfc6819).

Ex : la procédure de gestion et les informations de “session pour un utilisateur connecté" se basent sur le principe et la [technologie JWT](https://fr.wikipedia.org/wiki/JSON_Web_Token), standard actuel pour une exploitation Web sans état (stateless).

Ex : la brique applicative permettant la connexion côté frontaux est exclusivement basée sur une bibliothèque éprouvée : [Ember Simple Auth](https://github.com/simplabs/ember-simple-auth) (> 1800 stars sur GitHub).


### 1.2 - Des experts habitués plutôt que des talents curieux

Il y a des domaines où la curiosité, la créativité et l'esprit d'initiative sont les principales qualités requises pour réussir (monter une startup basée sur la blockchain ou la VR/AR). La sécurité n'en fait pas partie.

Dans le premier cas, l'enjeu est "d'imaginer les possibles". Dans le second, c'est de "prévenir les risques". Sans vision, on ne va nulle part ; mais sans sécurité, on ne va pas loin.

Le sujet est trop critique pour se permettre de faire mal les choses ou à moitié. Mieux vaut laisser faire ceux qui savent et qui pratiquent au quotidien et se consacrer à ses domaines de compétences et de valeurs ajoutées propres.

Ex : l'ensemble du SI de préproduction et de production est déployé, hébergé et monitoré par [Scalingo](https://scalingo.com/), dont la réactivité et la qualité continuent de se démontrer à chacune de leurs interventions.

Ex : Pix a consulté un expert attaché à la CNIL au bout de 5 mois de projet (novembre 2016).


### 1.3 - Des moyens adéquats plutôt que les contraintes du moment

La sécurité est un domaine qui peut très vite coûter cher (en argent, temps, communication, etc.) pour un retour sur un investissement indirect ou passif ("ça n'apporte rien, mais ça évite de perdre"). D'autant plus quand on fait appels à des experts (cf. point §1.2 ci-dessus).

Il arrive parfois que nous maîtrisions vraiment un sujet ou une solution, parce que nous l'avons étudié et déjà mis en oeuvre / exploité avec succès à plusieurs reprises. Cependant les ressources nous font défaut.

Le risque dans ce genre de situation est de repousser un sujet important jusqu'au dernier moment, en pensant gérer, et se retrouver finalement dépassé par les évènements. Là encore, la sécurité est un des domaines qui s'arrange le moins bien avec ce type de déviance et les conséquences peuvent être critiques ou définitives, en particulier pour une jeune entreprise comme Pix.


## 2. Less (personal data) is more (people tranquility)

La protection des données à caractère personnel est un sujet hautement complexe et critique à adresser. En particulier depuis l'entrée en application de la loi relative au RGPD.

Depuis le premier jour, le parti pris par Pix est de demander et de conserver le minimum de données personnelles pertinentes possibles.

Conserver un minimum de données personnelles rassure :
- les usagers de Pix, qui ne se sentent ainsi pas traqués
- les partenaires de Pix, qui craignent une fuite des données de leurs administrés / salariés qui pourraient nuire à leur image ou à leur business
- le GIP Pix, pour qui "moins on en sait, moins on a de responsabilités et mieux on se porte"

Chaque fois que nous implémentons une nouvelle fonctionnalité ou que nous établissons un contrat d'interopérabilité avec un partenaire, nous nous questionnons fortement sur les données en jeu : leur format, leur quantité, leur impact et leurs usages.

Pix n'a pas vocation à stocker des données à caractère :
- médical ou sexuel
- raciale ou ethnique
- religieux ou philosophique
- politique ou syndical
- etc...

Les données personnelles les plus sensibles stockées pour un utilisateur aujourd'hui sont :
- son email
- ses réponses
- ses résultats
- les signalements de tout utilisateur (connecté ou non, avec un e-mail ou non)

Ex : ce même système est repris dans la cadre de la connexion à Pix depuis d'autres systèmes de connexion. Plutôt que d'utiliser les identifiants uniques extérieurs, nous utiliserons et stockerons un ID Pix particulier rattaché, non pas au compte Pix directement, mais à un "MoyenDeConnexion" de type externe.

## 3. The closer to the data we are, the more safety features we deliver

La sécurité d'un SI est d'autant plus grande, qu'elle est prise en compte et mise en œuvre à tous les niveaux : technique, fonctionnel, organisationnel, etc.

Concernant le volet technique de Pix, le précepte que nous suivons est le suivant : "plus le code concerne les données, la base de données et leur accès, plus nous nous montrons prudents et multiplions les mécanismes de protection d'accès et d'exploitation de ces données".

Ex : Aussi bien les applications front que back sont accessibles exclusivement en TLS (Transport Layer Security) ; une redirection HTTP → HTTPS automatique est configurée pour chaque application.

Ex : Pour toutes nos applications Web, nous utilisons la même brique logicielle Open Source Ember Simple Auth, de façon que certaines pages ou interactions ne soient accessibles qu'à des utilisateurs connectés.

Ex : Chaque service de l'API est soumis à la stratégie de sécurité proposée par défaut par notre framework Hapi.js.

Ex : Chaque service de l'API est par défaut accessible à un utilisateur connecté seulement.

Ex : Certains services de l'API sont restreints à des utilisateurs connectés avec un rôle particulier.

Ex : La base de données utilisateurs (PostgreSQL) est chiffrée (pour toutes les tables & colonnes).
