---
layout: post
title: "Sélection et mise en œuvre d'une solution technique"
categories: architecture
excerpt: Comment nous identifions, sélectionnons et opérons une solution technique par rapport à nos besoins
cover:
    image: "selection-et-mise-en-oeuvre-d-une-solution-technique.jpg"
    source_title: Cesar Carlevarino Aragon
    source_url: https://unsplash.com/photos/NL_DF0Klepc
authors: jeremy_buget
---

## Introduction

À mesure que le service et l'organisation se développent, de nouveaux usages et enjeux opérationnels apparaissent, matérialisant de nouveaux problèmes, dont certains nécessitent la mise en œuvre de nouvelles solutions techniques.

La bonne compréhension des besoins et du contexte, l'identification de ces solutions et des éditeurs qui les proposent, leur réalisation / installation ainsi que leur bonne gestion représentent autant de points auxquels il convient de faire attention.

> ⚠️ Plus un changement (ligne de code, composant logiciel, service numérique) intervient en production dans l'urgence ou sous le coup de la précipitation, plus il aura tendance à y perdurer et consommer de précieuses ressources (temps, argent, énergie, sécurité psychologique), quelles que soient les intentions ou volontés à l'origine de son introduction.

Le présent article a pour but d'exposer et partager les réflexions que nous menons, ainsi que les critères que nous évaluons chaque fois qu'il est question d'ajouter un élement au SI, au niveau de la plateforme et des applications que nous développons ou des outils constituant notre environnement numérique de travail.

## Diagramme de décision

Depuis les origines de Pix, nous sommes animés par les mêmes valeurs : 
- la valeur et la qualité du service rendu aux utilisateurs au cœur de nos préoccupations et productions
- 

![image info](/assets/images/posts/selection-et-mise-en-oeuvre-d-une-solution-technique/decision-diagram.png)


## Les principaux critères de décision

### 1/ Build VS. Buy

### 2/ SaaS VS. IaaS

### 3/ Prime à la France et l'Europe

### 4/ Open Source VS. Licences propriétaires

### Autres critères 


## Conclusion

Choisir une solution par rapport à un besoin ou problème donné n'est jamais une tâche anodine, ni aisée. Les questions à se poser sont nombreuses, diverses et complexes. 

> Toute solution qui atterrit en production y demeurera aussi longtemps qu'il existera au moins un utilisateur régulier y trouvant son compte.

Définir une politique avec des guides, contraintes, principes directeurs, critères prioritaires ou secondaires et établir un processus complet de sélection d'une telle solution est une étape incontournable pour avancer plus rapidement, sereinement en préservant la cohérence et la consistance du SI.

Toutefois, aussi complet et éprouvé soit-il, le processus parfait n'existe pas et il arrive que nous devions faire certains choix en dépit de celui-ci. Cela doit rester exceptionnel (et assumé par l'organisation) sous peine de dénaturer, dégrader et déséquilibrer l'ensemble du SI.

Finalement, nous ne pouvons jamais être tout à fait sûrs de faire le bon choix. Ce n'est qu'après plusieurs mois ou années que l'on peut juger la pertinence d'une telle décision.

Chez Pix, nous faisons le choix stratégique de :

- développer nous-mêmes les services qui touchent à notre cœur de métier (gestion du contenu pédagogique, évaluation intelligente de compétences numériques, constitution profils de compétence, gestion des certifications, etc.)
- déléguer à des partenaires de confiance et avec lesquels nous entretenons une forte relation de proximité, les services connexes (IDP, APIM, mailing) ou annexes (analytics, forum, messagerie interne)
- en dernier ressort, exploiter nous-mêmes (auto-hébergement) les solutions open sources correspondant à notre besoin et nos valeurs / critères 

Au-delà même du résultat et des impacts sur le SI et l'organisation, le contexte peut évoluer, ainsi que les équipes, les besoins, les utilisateurs, le domaine, etc. En ce sens, une chose importante consiste à capitaliser sur le besoin, son contexte, ainsi que les raisons, les conditions et les conséquences ou impacts envisagés au moment du choix et de la mise en œuvre de la solution. En cela l'emploi d'[ADR (Architecture Decision Records)](https://en.wikipedia.org/wiki/Architectural_decision) représente un outil méthodologique et pédagogique intéressant.


## Annexes

## Checklist

### Analyse du besoin et du contexte

- [ ] Quel est le besoin à couvrir ? Dans quelle mesure est-il avéré (combien de personnes) ?
- [ ] Quelle est la criticité du besoin ? Dans quelle mesure accepte-t-on que la solution soit indisponible ?
- [ ] Possède-t-on en interne une solution susceptible de convenir (sans en détourner l'usage ou détériorer l'administration) ?
- [ ] Le besoin est-il cœur de métier pour le GIP Pix (Build) ou connexe / annexe (Buy) ?

Dans la suite de l'article, nous nous restreindrons au cas où nous ne sommes pas sur un besoin cœur de métier, et donc que nous devions sélectionner et opérer une solution managée ou SaaS.

- [ ] Quels sont les types de solutions (d'un point de vue des concepts théoriques, standards et best practices) en réponse au besoin ?
- [ ] Quels sont les concepts théoriques (à notre connaissance) que l'on souhaite ou s'attend à retrouver dans la solution ?
- [ ] Pour chacune, comment faut-il penser (architecture, code, infrastructure, outillage, process) son intégration par rapport à notre existant ?
- [ ] Quelles sont les solutions et acteurs **leaders** du marché ?


### Définition / sélection de la solution

- [ ] Existe-t-il une solution SaaS couvrant naturellement le besoin et mettant en œuvre les concepts / standards envisagés ?
- [ ] L'opérateur a-t-


- [ ] Comment envisage-t-on l'intégration - toujours d'un point de vue théorique - de la solution par rapport à l'existant ?
- [ ] À défaut de solution SaaS, existe-t-il une solution progiciel / Open Source qu'il est possible d'opérer ou faire opérer ?
- [ ] L'opérateur de la solution a-t-il son siège social en France ? en Europe ?
- [ ] Quelle est la réputation de l'opérateur ? Est-il connu et de quelle façon par notre entourage / réseau ?
- [ ] 
- [ ]
- [ ] Les données sont-elles hébergées en France ? En Europe ? Toutes ou partie ?
- [ ] Les données sont-elles répliquées sur des serveurs étrangers ?
- [ ] L'éditeur dispose-t-il d'un accord sur le traitement des données personnelles (DPA) ?
- [ ] Le service proposé par l'éditeur est-il référencé à l'UGAP ?
- [ ] Quel est le prix sur un mois / une anneé ? Est-ce que l'on reste sous la limite nécessitant un marché public ?
- [ ] Est-il possible de bénéficier de réduction par rapport à notre statut d'organisation à but non lucratif à visée éducative ?





## Build vs. Buy

La première chose est de définir si le problème à adresser ou le besoin à satisfaire touchent directement à ce que nous estimons faire partie de notre cœur de métier. De la réponse va découler une orientation critique par rapport au choix de la solution définitive, à savoir, opter pour un nouveau développement sur-mesure *in-house* ou bien déléguer la réalisation du service et sa mise en œuvre à une entité tierce. Il s'agit ici de notre implémentation / interprétation de la fameuse problématique "[Build vs. Buy](https://blog.octo.com/build-vs-buy/)", que l'on peut traduire en français par "logique de fabrication" contre "logique d'achat".

![image info](/assets/images/posts/selection-et-mise-en-oeuvre-d-une-solution-technique/build-vs-buy-pyramid.png)

Il existe plusieurs critères à évaluer pour trancher en faveur de l'une ou l'autre approche (cf. pyramide ci-dessus). Il est toutefois généralement admis que :

- l'approche Build est particulièrement adaptée pour un besoin qui relève du cœur de métier véritable et représente un avantage concurrentiel critique, unique à l'entreprise
- l'approche Buy est à réserver aux fonctionnalités ou services communs à toutes les sociétés
- entre les deux, il est possible d'opter pour une approche que l'on pourrait qualifier d'*hybride*, visant à mettre en place et administrer un progiciel

Au-delà de ce simple critère "cœur de métier concurrentiel ou non", il existe d'autres questions pertinentes qui permettent de se positionner, comme le synthétise l'arbre de décision ci-dessous issu de [cet article consacré au sujet](https://cultureconnectme.com/to-build-or-buy/) (en anglais).

![image info](/assets/images/posts/selection-et-mise-en-oeuvre-d-une-solution-technique/build-vs-buy-decision-tree.jpeg)

Dans le cas où l'on tombe du côté progiciel ou *buy*, de nouvelles questions se posent. Les points suivants traitent des sujets en question, en tentant de les ordonner par priorité ou criticité, tout en gardant à l'esprit que c'est la synthèse (teintée parfois d'une touche d'intuition ou de chance) de chacun des éléments unitaires qui guide notre choix final.

## 1. Le produit

Nous avons commencé par évoquer le poids de l'éditeur dans notre étude de sélection d'un outil informatique. Dans les faits et chronologiquement, on a généralement tendance à s'intéresser au premier car le second à attiré notre attention, par rapport à la recherche d'une solution pour un problème donné.

### L'existence de solution interne

Avant d'en arriver à explorer une solution et une entreprise nouvelles, nous nous posons d'abord la question de savoir si parmi tous nos outils (et ils commencent à être nombreux), l'un d'entre eux ne répondrait pas déjà au besoin.

Historiquement, nous préférons supprimer des choses (fonctionnalités, composants, contraintes, responsabilités) qu'en rajouter. Ajouter une brique au SI revient le plus souvent à ajouter de la complexité à celui-ci (et l'organisation). Autant que possible, c'est une finalité que nous voulons éviter.

Cependant, nous prenons bien garde à ne pas détourner un outil de son usage initial ou principal. Les quelques fois où nous l'avons fait, l'expérience a été peu concluante et il a fallu passer du temps à rechercher une nouvelle solution et à refaire le travail de mise en œuvre, communication, sensibilisation, etc.

> C'était par exemple le cas lorsque nous cherchions une solution d'édition de documents en ligne, collaborative et sécurisée.
>
> À un moment donné, nous avons eu l'idée d'utiliser l'outil de synchronisation des fichiers de [Cozy Cloud](https://cozy.io/fr/). Le programme est fait pour synchroniser *le plus régulièrement possible* un répertoire sur sa machine avec un compte d'un utilisateur unique et identifié dans le cloud. Au final, la synchronisation pour l'usage qu'on en faisait (beaucoup de fichiers édités par minute, sur une demi-douzaine de machines différentes) rendait la solution inefficace, peu fiable et gourmande en ressources (machine, réseau).
>
> Nous avons abandonné la solution au bout de plusieurs semaines, au profit d'une instance Nextcloud auto-hébergée, bien plus adaptée à nos besoins (mais aussi beaucoup plus lourde à administrer).

Une fois que nous avons fait le tour du SI et **si nous ne possédons pas la solution en interne nous recherchons une solution externe**.

### Les concepts sous-jacents

En général, il existe toujours plusieurs façons d'adresser un problème (le *why*). Une décision à acter en priorité est le type de solution (nature, concept, principes directeurs) recherchée (le *how*).

> Imaginons que l'on doive faire communiquer des pans de SI, est-ce que l'on souhaite opter pour approche [orientée service (SOA)](https://fr.wikipedia.org/wiki/Architecture_orient%C3%A9e_services), une solution à base d'ESB, ETL/ELT ou partir sur un style d'[architecture orientée évènement](https://fr.wikipedia.org/wiki/Architecture_orient%C3%A9e_%C3%A9v%C3%A9nements) / [Lambda Architecture](https://fr.wikipedia.org/wiki/Architecture_Lambda).

Certains concepts et modèles s'évaluent à l'aune des solutions qui les implémentent. Figer une réponse théorique à un problème pratique peut prendre du temps et nécessiter plusieurs tentatives infructueuses et allers-retours.

Quoi qu'il en soit, lorsque nous étudions une solution, nous évaluons aussi **les concepts sur lesquels elle s'appuie, à l'usage comme dans sa conception** et nous les confrontons à ceux que nous avions anticipés et envisagés.

### L'identification de solution

Une étape que nous n'avons pas encore citée est la découverte et l'identification des potentielles solutions. Sur ce point, rien de bien révolutionnaire :

- quelques recherches sur Qwant, DuckDuckGo ou Ecosia (et parfois Google ou Bing quand "vraiment" les réponses sont insuffisantes…)
- quelques sollicitations auprès de nos réseaux respectifs
- de la veille technologique régulière, plutôt généraliste

> Sur ce dernier point, nous partageons tous très régulièrement nos dernières découvertes en termes d'outils, services, concepts, pratiques, etc. Nous n'essayons pas tout ce que l'on voit, mais cet effort contribue à développer une culture technique forte, en plus d'éventuellement servir un jour.

### L'adéquation du produit avec nos besoins

Moment crucial du processus : la mise en correspondance des fonctionnalités proposées par la solution étudiée par rapport à nos propres besoins. Comme évoqué précédemment, ajouter un élément au SI représente une vraie prise de risque.

Sans surprise, **le premier critère à vérifier d'un point de vue produit est la couverture de nos besoins par des fonctionnalités cœur de métier pour le service concerné** et leur adéquation par rapport à nos standards et pratiques opérationnelles, culturelles ou organisationnelles.

Nous évaluons dans un second temps plusieurs autres points auxquels nous sommes naturellement sensibles :

- l'ergonomie
- la sécurité
- la performance (mais moins la tenue de charge)
- l'accessibilité (*)
- l'interopérabilité (la solution dispose-t-elle d'une API exploitable ?)
- l'administration de la solution
- la documentation
- la communauté

> (*) en tant qu'organisation éthique, et tout simplement en tant que structure professionnelle, nous nous devons de proposer à nos collaborateurs les outils adaptés à la bonne réalisation de leur mission, quelles que soient leurs conditions géographiques, environnementales, physiques, physiologiques, etc.

Nous ne passons pas des dizaines d'heures pour évaluer et comparer de façon scientifique et systématique chaque critère. Mais nous restons sensibles à l'ensemble de ces caractéristiques (qui sont autant de points que nous avons en tête quand nous développons nos propres services).

À partir de cette étape, nous nous trouvons généralement avec une *shortlist* (préselection en `FR_fr`) de solutions :

- soit très similaires
- soit très différentes

C'est alors qu'interviennent les autres critères de décision.

## 2. L'éditeur

### Siège et raison sociale

Les questions suivantes concernent l'éditeur, la nature de la solution logicielle qu'il fournit et sa façon de la gérer.

En tant que service public inter-ministériel et startup d'état de moins de 5 ans (avec un peu moins de 100 agents quotidien, tout type de contrat confondu), nous avons des convictions fortes quant au choix d'un partenaire et des produits ou services qu'il propose.

Un critère qui nous tient à cœur est la nature et la nationalité - au sens localisation du siège social - de nos fournisseurs. Ainsi, **nous privilégions les acteurs français 🇫🇷**, c'est-à-dire dont la siège social est domicilié en France.

Cela ne veut pas dire que nous excluons d'emblée toute entreprise ne satisfaisant pas à ces critères. Néanmoins, entre deux acteurs équivalents, mais administrativement situés dans des régions-monde différentes, nous trancherons en faveur de l'acteur français.

> À noter que nous ne considérons jamais un choix comme éternel. Ce fut l'un des critères qui nous a incité à changer de fournisseur de gestion des envois d'e-mail, cf l'[ADR-0006](https://github.com/1024pix/pix/blob/dev/docs/adr/0006-ajout-du-support-de-sendinblue-pour-le-mailing.md) et la [PR-997](https://github.com/1024pix/pix/pull/997).

Parfois, il n'existe pas (ou plus, faute de concurrence létale ou rachat par des entreprises étrangères) d'éditeur français en capacité de fournir une solution correspondante à nos besoins. Nous privilégions alors des **sociétés et services Européens 🇪🇺**. Ce n'est qu'en dernier recours que nous acceptons de nous tourner vers des sociétés hors UE.

Au-delà de la fibre patriotique (française ou européenne) qui nous anime, il est important pour nous de considérer, via ce critère, le respect et l'adéquation de l'entreprise avec les lois et règlements auxquels nous sommes nous-mêmes confrontés (RGPD, RGAA, RGS, etc.).

Sur bien des aspects (accessibilité, juridique, sécurité, résilience, etc.), la performance de la chaîne opérationnelle dépend de celle du maillon le plus faible. Notre conformité avec le RGPD ou le RGAA, par exemple, dépend de celle de chacun de nos partenaires et fournisseurs. Il est donc crucial pour nous de **sélectionner des entreprises qui vivent les mêmes contraintes règlementaires**.

### Viabilité

Un autre point important concernant un potentiel partenaire est sa viabilité. La contractualisation avec une entreprise tierce et la mise en production du service qu'elle propose est loin d'être une tâche anodine. C'est souvent un investissement conséquent, accompagné d'un processus long ou lourd, et qui va nécessiter des efforts de gestion de notre côté (opérationnelle, commerciale, juridique) pour une longue période.

Avant de nous engager avec une société, nous devons nous assurer qu'elle a les reins assez solide pour supporter notre collaboration, mais aussi notre développement.

De fait, nous déclinons toute solution et leur éditeur pour lesquelles nous n'avons ou ne ressentons pas de garanties suffisantes. Par exemple, il nous est arrivé de ne pas donner suite à des entreprises dont la promesse commerciale (sur leur site vitrine ou en brochure) nous faisait de l'œil, mais qui accumulaient des détails suspects :

- sensation qu'il s'agit d'une [*One Person Company*](https://www.quora.com/What-is-one-person-company-1) au travers des mentions légales ou CGU/CGV
- communauté discrète voire quasi nulle
- dans le cas d'une offre basée sur un produit open source, nombre de stars + contributions + contributors
- si un blog est présent, le nombre d'articles, de rédacteurs, de commentaires, la fréquence, etc.

Nous ne demandons pas (encore) de bilan de société sur plusieurs années. Mais nous vérifions rapidement la santé générale de l'entreprise. Nous étudions les chiffres (date de création, nombre de salariés, évolution, CA, …) quand ils sont indiqués ou disponibles.

### Réputation et relationnel

Dans la veine du point précédent, nous nous appuyons sur notre réseau et les diverses communautés (BetaGouv, Alumnix, Tech.Rocks, etc.) dans lesquelles nous baignons pour nous faire un avis plus précis encore. En plus de nous enquérir de la qualité des services proposés, nous nous intéressons à tous les aspects "relation commerciale". En tant que structure agile, pour nous, la qualité de la *collaboration* passe bien avant celle de la *négociation* 👫.

Il nous arrive d'aller jusqu'à éplucher les divers comptes de l'entreprises sur les réseaux sociaux, pour se faire une idée de la tonalité de la communication, des messages, de leur fréquence, transparence, sincérité, ainsi que les commentaires de leur clients ou utilisateurs.

Pour finir sur ce point, et bien que ça semble être du bon sens, nous n'hésitons pas à rentrer en contact directement avec l'entreprise étudiée pour échanger de vive voix, demander des détails, voire récolter des infos qui ne sont pas disponibles sur la documentation en ligne. C'est à la fois simple, et en même temps, ça demande un petit peu d'effort (notamment d'organisation) et de patience. Par les temps d'automatisation et d'autonomisation à tout-va, c'est peut-être un réflexe qui se perd.

> 💡 Bon à savoir : nous ne sommes pas complètement fermés à l'idée de donner sa chance à des entreprises jeunes ou au début de leur aventure. Chaque fois que nous avons tenté l'aventure (Scalingo, Baleen, Gravitee), les résultats ont été très positifs pour le GIP Pix et pour nos utilisateurs, au niveau opérationnel mais aussi commercial ou tout simplement relationnel. L'avantage de travailler avec une société de taille modeste est une meilleure écoute, ainsi qu'une réelle proximité et réactivité. Ce faisant, chacun de nous s'inscrit vraiment dans une approche gagnant-gagnant.

### Culture, organisation et gestion de produit

C'est à l'occasion de ce type d'échanges qu'on peut justement en apprendre plus et vraiment découvrir notre prospet partenaire, en particulier sur les aspects de culture, d'organisation interne et de gestion de projet.

Nous sommes fiers et surtout heureux de pouvoir délivrer très régulièrement de la valeur véritable à nos utilisateurs. Par extension, nous sommes naturellement plus enclins à collaborer avec des organisations qui partagent ce trait de culture. Nous savons les efforts et investissements que cela nécessite. C'est le gage (peut-être naïvement ou à tort) de partager un langage, des pratiques et une culture communes et, d'une certaine façon, l'assurance qu'en cas de difficulté ou situation exceptionnelle, nous saurons réagir en "équipe" efficacement, rapidement et confortablement. #QuiSeRessembleSAssemble

Il arrive qu'avec le temps et au gré de nos succès communs, nous finissions mêmes par échanger régulièrement sur des questions stratégiques, techniques, opérationnelles, organisationnelles, etc. Au-delà du service fourni ou de la prestation, c'est un moyen de s'enrichir et de s'aider mutuellement.

## 3. Les données

Nous ne le répèterons jamais assez : **la sécurité des données et services délivrés, ainsi que le respect de la vie privée de nos utilisateurs sont des enjeux sacrés pour Pix**. Pas seulement pour le côté "risques" qui y sont liés, mais aussi (et surtout) "par principe". Contribuer à montrer l'exemple et servir de porte-étendard sur ces sujets sensibles en plein essor (*NDLA : "il était temps !"*) fait partie des objectifs secondaires qui nous motivent au quotidien et que l'on se fixe.

## SaaS vs. PaaS vs. IaaS

## Tarification et facturation

## Autres questions
