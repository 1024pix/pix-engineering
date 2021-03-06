---
layout: post
title:  "Poisson d'avril !"
date:   2021-05-11 10:10:42 +0200
categories: architecture
excerpt: La vie et la mort d'une blague pour le 1er Avril.
cover:
  image: "poisson-avril.jpg"
authors: francois_de_metz
---

Parce que nous aimons rire, pour le 1er avril, nous avons décidé de faire une blague à nos utilisateurs. J'aimerais vous raconter comment cette blague est née, codée et déployée, car à l'échelle de Pix, une blague doit être réalisée sérieusement.

Tout a été imaginé et implémenté en une après-midi du 31 mars 2021, lors d'un "Tech Time". Le "Tech Time" c'est une après-midi toutes les 2 semaines dans laquelle l'équipe de développement se retrouve pour travailler sur des sujets non fonctionnels, et pas forcément en rapport avec Pix.

#### Prise de décision

Nous nous retrouvons vers 14h30 pour discuter de quoi faire. Rapidement, nous envisageons de faire quelque chose avec le compteur de Pix, élément central de l'interface. Décision est prise de le faire inexorablement baisser.

Voilà à quoi ressemble le compteur à la base :

![Le compteur de Pix](/assets/images/posts/poisson-avril/compteur-pix.png)

#### Implémentation

Nous implémentons rapidement la première itération en décrémentant le compteur avec comme valeur minimale -999. Tout se passe en TDD et en mob programming, par plage de 15 minutes. Tests unitaires, tests d'intégration tout y passe.

Nous nous rendons compte que nous devons gérer le cycle de vie de la blague. Il va falloir l'activer le 1er avril mais surtout l'arrêter dans la nuit. Pas question de déployer une version spécifiquement pour cela. Nous utilisons alors un mécanisme présent dans l'API de bascule de fonctionnalités (feature toggle) pour pouvoir le faire à chaud.

Les comportements sont testés, l'API est modifiée et nous faisons [la pull request sur le dépôt](https://github.com/1024pix/pix/pull/2794).

**Le spectacle**

A 17h, c'est l'heure du spectacle. Nous montrons le résultat à l'équipe. Le voici:

<video src="/assets/images/posts/poisson-avril/spectacle.mp4" controls></video>

Hilarant n'est-ce pas ?

#### La communication interne

Vers 18h, il est temps d'informer le reste de Pix de cette "fonctionnalité", et surtout de valider que nous pouvons la déployer et l'activer.

Pour ne pas perturber les sessions de certifications du matin, il est décider de n'activer le poisson que le 1er Avril à 16h30.

#### Le déroulé

- 31 mars - 19h30 : La PR est fusionnée.
- 1er Avril - 10h20 : Une mise en recette est faite.
- 1er Avril - 17h10 : La mise en production est effectuée. La blague est ensuite activée en ajoutant la variable d'environnement `FT_IS_APRIL_FOOL_ENABLED=true`.
- 1er Avril - 22h30 : La variable d'environnement est supprimée. La blague n'est plus active :(
- 6 avril : la blague est supprimée du code

#### Conclusion

Nous remercions tout d'abord le support qui a géré les retours d'utilisateurs, certains amusés, d'autres moins.

**Qu'avons-nous appris ?**

Si la fonctionnalité a été rapidement codée, la plus grosse partie du boulot a été de faciliter le déploiement pour que tout se passe bien. La bascule à chaud de fonctionnalité est bien précieuse pour cela.

Imaginer, coder, tester et déployer une fonctionnalité non prévue en ~24h, c'est possible, et c'est bien sympa.

Si tu aimes le service public, coder et surtout t'amuser, [nous recrutons](https://www.welcometothejungle.com/fr/companies/pix).
