---
layout: post
title:  "Tester unitairement les composants sous Embroider"
date: 2023-05-22 12:27:00 +0200
categories: tests
excerpt: "Découvrez comment faire des tests unitaires de vos composants Glimmer dans votre application Ember.js utilisant Embroider."
cover:
  image: "tester-unitairement-les-composants-sous-embroider.png"
authors: vincent_hardouin
---

Nous avons récemment migré nos applications Ember.js sous [Embroider](https://github.com/embroider-build/embroider/),
le nouveau builder pour Ember.js. Embroider offre une fonctionnalité intéressante appelée "route splitting",
qui permet d'envoyer uniquement les fichiers JavaScript nécessaires pour une ou un groupe de routes.
Cela réduit la taille du JavaScript téléchargé par les utilisateurs, à condition que le découpage des routes soit bien pensé.

Cependant, l'activation de cette fonctionnalité nécessite plusieurs options, dont `staticComponents`,
qui a pour effet de résoudre tous les composants nécessaires et de les inclure dans la construction finale.

En activant cette option et en suivant [le guide de migration pour les composants helpers](https://github.com/embroider-build/embroider/blob/main/docs/replacing-component-helper.md),
les tests unitaires de nos composants ne passaient plus.

## Le défi des tests unitaires avec Embroider et les composants Glimmer

Lors de la migration vers les composants Glimmer, nous nous étions rendu compte que Glimmer ne supportait plus nativement les tests unitaires,
nous avions utilisé une parade proposée par la communauté sur le [Discord d'Ember](https://discord.gg/emberjs),
que vous pouvez retrouver en détail sur [cet article](https://timgthomas.com/2019/11/unit-testing-glimmer-components/) et dont voici la solution :

```javascript
import { getContext } from '@ember/test-helpers';
import GlimmerComponentManager from '@glimmer/component/-private/ember-component-manager';

export default function createComponent(lookupPath, named = {}) {
 const { owner } = getContext();
 const { class: componentClass } = owner.factoryFor(lookupPath);
 const componentManager = new GlimmerComponentManager(owner);
 return componentManager.createComponent(componentClass, { named });
}
```

Qui permet dans un test unitaire de faire :

```javascript
const component = createComponent('component:component-name', { argument });
```

Cette solution est désormais obsolète, car avec Embroider et l'option `staticComponents`
la méthode `factoryFor` ne retourne rien.
Cela parait cohérent avec la volonté d'Embroider qui souhaite créer des builds statiques analysables.

## Une nouvelle solution avec importSync de @embroider/macros

La nouvelle solution que nous utilisons se sert  d'`importSync` proposé par `@embroider/macros`,
qui permet à Embroider de signaler l'existence d'un module et de l'inclure dans le build.

Voici donc le code avec la modification :

```javascript
import { getContext } from '@ember/test-helpers';
import GlimmerComponentManager from '@glimmer/component/-private/ember-component-manager';
import { importSync } from '@embroider/macros';

export default function createComponent(lookupPath, named = {}) {
 const { owner } = getContext();
 const componentClass = importSync(`../../components/${lookupPath}`).default;
 const componentManager = new GlimmerComponentManager(owner);
 return componentManager.createComponent(componentClass, { named });
}
```

Et son usage

```javascript
const component = createComponent('component-name', { argument });
```

Grâce à cette simple modification, nous pouvons à nouveau continuer de tester unitairement nos composants Glimmer et
commencer à utiliser les fonctionnalités proposées par Embroider.
