---
layout: post
title:  "Suivi de migration avec Steampipe"
date:   2022-09-20 10:10:42 +0200
categories: bdd
excerpt: "L'histoire d'un tableau de bord qui a permis de suivre les différentes étapes de la migration de notre base de données."
cover:
  image: "dashboard-biging.png"
authors: francois_de_metz
---

Le 23 juillet 2022 à 22h et jusqu'au 24 juillet à 17h, Pix était en maintenance. La raison ? Nous avons profité de la période de calme de l'été pour migrer l'identifiant primaire de l'une des tables de notre base de données de int à bigint. Cela fait suite à la [précédente migration d'août dernier]({% post_url 2021-08-31-modifier-type-clef-primaire-non-referencee-vers-biginit %}).

Dans ce genre de moment, tout le monde doit être aligné. En effet pour que la migration se fasse, il faut s'assurer que l'ensemble des applications impactées soient en maintenance, que la base de données soit dans le bon état, lancer le script de migration, attendre (longtemps), vérifier le bon résultat, et relancer les applications.

Pour tout cela il est nécessaire d'avoir de bons outils. Des outils nous en avons justement à foison, entre le monitoring externe de nos applications [Freshping][], [Datadog][] pour les logs des applications, [Scalingo][] notre hébergeur et [Confluence][] pour la documentation.

Tous ces outils, associés à 6 personnes qui doivent se relayer pas toujours à l'aise avec les outils de suivi de la production, rendent l'alignement complexe.

Pour résoudre ce problème nous avons donc utilisé un nouvel outil, [Steampipe][], pour les gouverner tous.

Nous utilisons déjà Steampipe pour vérifier l'état de la production tous les jours, mais c'était notre première utilisation des dashboards.

Nous l'avons construit pour permettre un suivi chronologique de la migration:

1. La mise en maintenance des applications Pix impactées
2. Le suivi de la migration
3. La réouverture

Le dashboard agit comme une tour de contrôle de la migration. En un coup d'oeil, nous savons où nous en sommes.
Le code du dashboard est, comme beaucoup de choses chez Pix, [accessible sur GitHub](https://github.com/1024pix/steampipe-dashboard-bigint).

Trêve de bavardage, à quoi il ressemble ?

Au début tout est rouge:

![](/assets/images/posts/dashboard-bigint/rouge.png)

Et puis une fois la mise en maintenance effectuée, cela passe au vert (vous avez remarqué les jolies icônes ?)

![](/assets/images/posts/dashboard-bigint/vert.png)

Tout est éteint certes, mais est-ce que nous n'avons pas oublié des applications qui auraient des connexions ouvertes sur la base ?

![](/assets/images/posts/dashboard-bigint/open.png)

Ensuite on lance le processus de migration:

![](/assets/images/posts/dashboard-bigint/progress.png)

Au bout de quelques heures, on y est ?

![](/assets/images/posts/dashboard-bigint/presque.png)

OUI

![](/assets/images/posts/dashboard-bigint/oui.png)


## Sous le capot

Coté hébergement, comme pour tout le reste, nous l'avions déployé sur Scalingo grâce au [buildpack steampipe][]. Comme nous manipulions des informations de production, il était sur la région osc-secnum-fr1.

La partie suivie de la mise en maintenance utilisait le [plugin net][], en vérifiant le code HTTP 503 des applications. Pour les applications non directement accessibles depuis internet, nous avons utilisé le [plugin scalingo][] pour vérifier que tous les conteneurs étaient éteints. La vérification de la désactivation du monitoring a été faite avec le [plugin freshping][].

Avant de lancer la migration, nous avions le nombre de connexions ouvertes a PostgreSQL affichées. Utile pour savoir si tout avait bien été éteint. La récupération se faisait avec le [plugin datadog](), qui récupérait les infos de [pix-db-stats][].

Le suivi de la migration était plutôt minimale, parce que c'est PostgreSQL qui faisait le travail, sans réelle possibilité de connaitre de suivre précisémenet la progression. Mais nous pouvions monitorer les différentes étapes du script de migration, et ce sur les 2 bases à migrer simultanément. Pour cela le [plugin datadog][] regardait les logs générés par les conteneurs pour afficher l'étape en cours.

## Conclusion

La migration s'est bien déroulée, et nous sommes contents d'avoir pu expérimenter les tableaux de bord Steampipe. Sa capacité à interroger différents services, de les mixer et d'avoir une jolie interface permet de démarrer très rapidement et de rajouter au fur et a mesure des informations.

Des choses étaient tout de même perfectibles, comme le suivi de la migration qui était un peu fragile, ou bien certaines étapes de la mise en maintenance qui n'étaient pas visibles dans Steampipe, comme l'arrêt de l'infrastructure data sur OVH (par manque de temps principalement).

Si nous espérons ne pas avoir souvent à mettre en maintenance tout pix, avoir un outil qui nous permet de centraliser toutes les informations nous sera certainement utile à l'avenir avec l'expérience accumulée cette fois-ci.

[Freshping]: https://www.freshworks.com/website-monitoring/
[Datadog]: https://www.datadoghq.com/
[Scalingo]: https://scalingo.com/
[Confluence]: https://www.atlassian.com/software/confluence
[Steampipe]: https://steampipe.io/
[buildpack steampipe]: https://github.com/francois2metz/steampipe-buildpack
[plugin net]: https://hub.steampipe.io/plugins/turbot/net
[plugin scalingo]: https://hub.steampipe.io/plugins/francois2metz/scalingo
[plugin freshping]: https://github.com/francois2metz/steampipe-plugin-freshping
[plugin datadog]: https://hub.steampipe.io/plugins/turbot/datadog
[pix-db-stats]: https://github.com/1024pix/pix-db-stats
