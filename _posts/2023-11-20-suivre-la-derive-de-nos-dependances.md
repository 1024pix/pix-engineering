---
title: "Suivi de la dérive de nos dépendances"
lang: fr
date: 2023-11-23
excerpt: "Ou comment voir le retard des mises à jour à faire sur l'ensemble de nos applications."
authors: francois_de_metz
cover:
  image: "suivi-dependance.png"
  source_url: https://cdn.midjourney.com/96f813ae-de0e-4ce9-852c-631e8916f792/0_2.webp
---

## Le constat

Le suivi de nos dépendances logiciels n'a jamais été un sujet suivi chez Pix. Les mises à jour se sont fait donc au bon vouloir de quelques personnes.

Dans le passé, une petite équipe avait été créée pour mettre à jour Ember sur nos applications clientes. Cela fonctionne bien par à-coups, mais ne règle pas le problème de suivi.

C'est pourquoi [Renovate][] a été mis en place. Renovate est présent sur tous nos dépôts et nous pousse en permanence des mises à jour de nos dépendances. Avec le "[Dependency dashboard](https://docs.renovatebot.com/key-concepts/dashboard/)" ([voir celui de 1024pix/pix](https://github.com/1024pix/pix/issues/5637)), nous pouvons avoir les différentes mises à jour en attente. Cela nous donne une idée du retard mais cela reste très grossier.

C'est pourquoi lors des Tech Days 2023 (du travail technique réalisé pendant l'été), une équipe s'est chargée de remettre de l'effort sur les mises à jour de dépendances. Le travail a été de passer à node 18 sur notre application principale et de configurer Renovate pour que tout roule.

Le besoin de quantifier d'une manière commune le retard de mise à jour des dépendances sur les différents projets nous a amené à nous intéresser à une métrique intéressante, [libyear][].

## Libyear à la rescousse

En bref [libyear][] permet de quantifier l'âge de l'ensemble des dépendances.

En calculant cette métrique sur nos différents projets, nous avons une mesure **comparable** pour **constater** et **décider** des actions de mise à jour.
C'est pourquoi nous avons développé [dependency-drift-tracker][] qui nous permet de connaitre la dérive actuelle de nos dépendances, de voir l'évolution et de comparer ce retard avec les autres.

En plus du retard de nos dépendances, nous affichons également la mesure en années depuis la dernière release, appelée Pulse. Cela permet de repérer des dépendances qui sont devenues obsolètes ou dépréciées. Cela nécessite une analyse un peu plus poussée manuelle.

## A quoi cela ressemble ?

![L'interface de dependency-drift-tracker](/assets/images/posts/suivi-dependance/dependency-drift-tracker.png)

À gauche, en un clin d’œil, on voit les dépôts suivis avec l'information du retard. Ensuite dans le panneau principal, 2 métriques sont affichées, le retard et et le pulse.

Les 2 graphs suivants permettent de suivre l'évolution de ces chiffres à travers le temps.

Enfin le tableau final affiche le résultat du dernier lancement de libyear avec les informations de dépendances individuelles.

Les données sont rafraîchies tous les jours. Un retour suffisamment régulier pour valoriser le travail accompli la veille, et planifier la suite.

## Conclusion

Rendre visible notre retard nous a permis de nous améliorer sur le suivi des versions. Voir la baisse de la courbe après une mise à jour est toujours un plaisir. Même si, d'une manière continue et semble t'il inexorable, la courbe remonte suite aux mises à jour des dizaines de dépendances utilisées par nos applications.

Curieux·ses, voici le [suivi des applications Pix](https://1024pix.github.io/dependency-drift-tracker/).

## Essayez-le

Nous avons rendu le code de suivi générique pour être utilisable par n'importe quel projet javascript.
Vous pouvez facilement gérer les fichiers de suivi manuellement via [dependency-drift-tracker][], ou encore plus simplement avec l'action GitHub [dependency-drift-tracker-action][].

Et lisez la [documentation](https://github.com/Dependency-Drift-Tracker/dependency-drift-tracker?tab=readme-ov-file#usage).

[renovate]: https://www.mend.io/renovate/
[libyear]: https://libyear.com/
[dependency-drift-tracker]: https://github.com/Dependency-Drift-Tracker/dependency-drift-tracker
[dependency-drift-tracker-action]: https://github.com/Dependency-Drift-Tracker/dependency-drift-tracker-action
[dependency-drift-status]: https://github.com/Dependency-Drift-Tracker/dependency-drift-status
