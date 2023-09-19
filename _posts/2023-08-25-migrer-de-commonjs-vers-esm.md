---
layout: post
title:  "Migrer de CommonJS vers ECMAScript Modules #1"
date: 2023-08-25 09:49:00 +0200
categories: javascript
excerpt: "Récit d'une aventure : migrer 400kLoc NodeJs en deux mois - #1: les modules"
cover:
  image: "cjs-to-esm.png"
  source_url: https://raw.githubusercontent.com/wessberg/cjstoesm/master/documentation/asset/logo.png
authors: pierre_top
---

## Y'a quoi au menu ?
Cet article partage avec vous deux mois d'efforts (et de doutes, aussi) pour migrer une base de code NodeJS de 400 kLoC. Spoiler : nous y sommes arrivés !

Vous y trouverez de l'histoire, de la syntaxe, des considérations techniques et organisationnelles, de l'outillage et du monitoring : il y en a pour tous les goûts.

Voilà, en entrée, le résumé du challenge à accomplir :
- [comprendre](#une-histoire-de-modules) ce qu'est un système de module ;
- se souvenir [de ce qu'il en est](#les-modules-côté-javascript) en Javascript ;
- savoir si ESM [peut t'être utile](#pourquoi-migrer-en-esm-) ;
- comprendre [la syntaxe ESM](#un-peu-de-syntaxe) ;
- comment [modifier le code](#modifier-le-code) (la [syntaxe](#un-peu-de-syntaxe) est un pré-requis).

Si vous êtes déjà familier avec tout ça, attendez plutôt la sortie de l'article [en plat principal](https://github.com/1024pix/pix-engineering/pull/42), à savoir comment nous avons mené la migration.

## Une histoire de modules

Dans la plupart des langages de programmation (C, Java, JavaScript...), le fichier de code (source) est un élément important. Il est implémenté dans le système de fichiers de l'OS, par un fichier texte. Il y a quelque chose de fondamentalement réconfortant à cela. Utiliser un langage qui stocke le code source dans des fichiers aux noms cryptiques et au contenu binaire, c'est une perte d'autonomie.

Pourquoi ? Parce que ranger des fichiers dans des dossiers est une métaphore puissante : elle permet de regrouper ce qui se ressemble en traçant des frontières. C'est la même chose qui se passe au niveau en-dessous lorsqu'on crée des fonctions.

> Aside from the computer itself, the routine is the single greatest invention in computer science. (...) Create a routine to hide information so that you won't need to think about it. (...) Without the abstractive power of routines, complex programs would be impossible to manage intellectually.

in Steve McConnell, Code Complete - Chapter 7 : High-quality routine

En théorie informatique, ce genre de découpage (de décomposition) porte le nom de [module](https://en.wikipedia.org/wiki/Modular_programming), un terme déjà utilisé en [construction](https://www.cnrtl.fr/definition/module) pour exprimer la capacité à bâtir un système en assemblant des composants. La réutilisation de code, sous forme de librairie, en fait partie.

## Les modules côté Javascript

La notion de module est assez large en Javascript : on désigne parfois la même chose par librairie, package, script, balise `<script>`, module, fichier ... Revenons en arrière pour mieux comprendre.

### Tout débute dans un navigateur

Au début de Javascript (1995), il n'y avait pas de système de module natif : tout le code était contenu dans la balise `<script></script>`. Pour que les "pages web" (contenu statique) puissent devenir des applications web (contenu dynamique), des fonctionnalités génériques (ex : appeler une API REST) sont distribuées sous forme de librairies Js. Elles pouvaient être importées depuis un fichier via l'attribut `src` de la balise `script`, par exemple `<script src="my-library.js">`. Le code de l'application (composant) pouvait aussi être modularisé ainsi.

### Un petit tour côté serveur

Le passage de Javascript côté serveur (2009) changea l'approche.

Dans NodeJS, les libraires (packages) :
- déclarent leur interface (API) avec le mot-clef `module.exports` (notion de module) ;
- sont enregistrées par l'application dans le fichier `package.json` ;
- sont installées par npm dans le fichier `node_modules` (encore le module) ;
- sont importées avec le mot-clef `require`.

Les composants internes de l'application utilisent les mêmes mot-clefs `require` et `module.exports`. Il n'y a pas de différence visible entre l'import d'une libraire et d'un composant, à part le fait qu'une libraire est mentionnée par son nom et le composant par son chemin.

```javascript
// library
const { pick } = require('lodash');
// component
const { controller } = require('./lib/application/certification/certification-controller');
````

### Happy end ?

Maintenant que nous sommes familiers avec le concept de modules dans Javascript, nous arrivons enfin à ESM.

Actuellement, il y a plusieurs systèmes de modules en Javascript : AMD, CJS, ESM, UMD.

Le système CommonJs (en abrégé CJS) est [le plus utilisé](https://github.com/wooorm/npm-esm-vs-cjs) côté serveur, car :
- CJS était fourni avec NodeJs ;
- il n'existait pas de système de module natif, partagé par les navigateurs et NodeJs.

En 2015, le comité chargé du langage met un terme à cette situation, à savoir :
- le comité TC39 (chargé de Javascript)
- de l'organisation de normalisation ECMA
- publie dans la version 2015
- la spécification ECMAScript module, en abrégé ESM.

Cinq ans plus tard, en 2020, les navigateurs web et NodeJs (v12) l'ont implémenté de manière stable.

## Pourquoi migrer en ESM ?

La base de code Pix, créée en 2011, utilise le format CJS. Si le format qu'elle utilise est éprouvé, pourquoi migrer pour un système qui n'a que trois ans de support officiel ? La décision tient à une raison : faciliter le passage à Typescript.

Si vous n'êtes pas dans cette situation et cherchez ce que ESM pourrait vous apporter, nous conseillons [cet article](https://webreflection.medium.com/cjs-vs-esm-5f8b90a4511a), écrit par un membre de [l'équipe](https://github.com/nodejs/modules) qui a implémenté ESM dans NodeJS
>  CJS vs ESM is not a war, rather a topic to talk even more about, once constraints and respective features are clear, as opposite of just taking a side out of habits, or effort needed, to move on (to ESM).

Voilà tout de même les avantages d'expérience de développement après quelques mois d'utilisation :
- [mode strict](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Strict_mode) actif ;
- autocomplétion des imports par les IDE ;
- vérification [statique](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/named.md) des imports, absente en CJS;
- syntaxe moins permissive, qui améliore la lisibilité du code.

Pour illustrer la syntaxe, voilà une version valide en CJS.
```javascript
const Ham = require('./ham').Ham;
const carrot = require('./garden').digCarrot();
const potatoes = require('./field').harvestPotatoes(require('./tools').basket);

/* some code */

if( some_condition()){
   require('./orchard').spray();
}

const omelette = cook(require('./egg'));

/* some code */

module.exports = {
  ham: new Ham(),
  someFunction : require('./spam').someFunction
}
```

Et voilà la version équivalente en ESM :
- les imports sont visibles en haut du fichier ;
- l'interface (= les exports) est claire.

```javascript
import { Ham } from './Ham.js';
import egg from './egg.js';
import { someFunction } from './spam.js';
import { digCarrot } from './garden.js';
import { spray } from './orchard.js';
import { harvestPotatoes } from './field';
import { basket } from './tools';

const potatoes = digPotatoes(basket);
const carrot = digCarrot();

/* some code */

if( some_condition() ){
   spray();
}

const omelette = cook(egg);

/* some code */

const ham = new Ham();

export {
   ham,
   someFunction
}
```

## Un peu de syntaxe

Pour que vous puissiez suivre la démarche de migration, il est important d'avoir les bases de la syntaxe ESM. Partons du principe que vous connaissez [la syntaxe CJS](https://apprendre-nodejs.fr/v1/chapter-04/index.html#modules).

Tous les composants (nombre, objets, fonctions, classes) peuvent être exposés dans les modules ESM. Ils portent un nom d'export. Celui-ci peut être remplacé par un autre lors de l'import dans un autre module. Il est aussi possible de supprimer leur nom à l'export, pour le remplacer par le nom `default`. Il ne s'agit pas à proprement parler d'export anonyme, simplement d'un export nommé `default`, aussi appelé export par défaut.

Les imports et exports doivent être à la racine du fichier (`top-level`).
Ils sont interdits dans une structure de contrôle ou une fonction.

Les imports doivent figurer en début de fichier.
Leur syntaxe est la suivante, du plus simple au plus complexe :
  - `import from './module.js'` : si l'on ne veut pas récupérer d'import, uniquement exécuter du code
  - `import module from './module.js'` : pour récupérer l'export par défaut et le nommer `module`
  - `import { foo, bar } from './module.js'` : pour récupérer certains exports nommés
  - `import { foo , bar as baz } from './module.js'` : même chose, en les renommant
  - `import * as module from './module.js'` : pour récupérer tous les exports nommés dans un objet

La syntaxe des exports est la suivante, du plus simple au plus complexe :
  - `export foo;` : exporter un objet, peut être invoqué plusieurs fois
  - `export default foo` : export par défaut (l'objet exporté perd son nom `foo` pour prendre celui de `default`)
  - `export { foo, bar }` : export de plusieurs objets
  - `export { foo , bar as baz }` : même chose, avec renommage
  - `export { foo, bar } from './module.js` : vous avez bien lu, on importe les exports nommés et on les exporte (ré-export)
  - `export * from './module.js` : même chose, pour sélectionner tous les exports nommés

Si vous êtes curieux de connaître l'implémentation de ESM, [cet article](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/) est une excellente introduction sur les modules en général et ESM en particulier.

## Modifier le code

### Une solution simple

Le passage de CJS à ESM concerne tous les fichiers avec au moins un import/export, autrement dit tous les fichiers.
Étant donné la volumétrie, une modification de code manuelle est exclue. Quelles sont les solutions pour modifier du code ?

Si les modifications sont mineures, par exemple supprimer le `s` de `exports`, les outils quotidiens du développeur suffisent : soit IDE (Find/Replace avec expressions régulières), soit un script bash utilisant find, grep, sed. Si l'on veut éviter d'écrire du bash, une solution est de modifier le fichier avec NodeJs, en utilisant la [librairie I/O](https://nodejs.dev/en/learn/reading-files-with-nodejs/) standard. Les lignes de code sont des chaînes de caractère comme les autres, après tout.

L'affaire se complique si la transformation modifie la syntaxe et pas seulement un mot-clef.
```javascript
// CJS
const { Ham } = require('./ham');
// ESM
import { Ham } from './Ham.js';
```

Car, même si transformer `const { <TOKEN> }` en  `import { <TOKEN }` semble simple, il faut aussi éviter les effets de bord sur des expressions qui y ressemblent, mais qui n'en sont pas, par exemple les imports nommés non sélectifs.
```javascript
// CJS
const egg = require('./egg.js');
// ESM
import egg from './egg.js';
```

Et que dire si la transformation est multi-lignes ?
```javascript
// CJS
const ham  = new (require('./ham')).Ham();
// ESM
import { Ham } from './Ham.js';
const ham = new Ham();
```

Et encore, ici les lignes sont consécutives, alors qu'il est indispensable en réalité de pouvoir modifier les exports, qui peuvent être n'importe où dans le fichier.

Bref, il nous fallait une autre solution. Nous avons cherché comment d'autres avaient fait, car nous ne sommes pas les seuls à vouloir migrer en ESM. Nous avons constaté que leurs solutions ([celle-ci](https://github.com/azu/commonjs-to-es-module-codemod#readme) ou [celle-là](https://github.com/wessberg/cjstoesm)) ne fonctionnaient pas sur notre codebase, mais elles nous ont mis sur la voie.

Entrons dans le monde de l'AST, dirigé par les `codemods`.

### Une solution complexe et puissante : les codemods


#### L'AST
Les outils dédiés à la modification de code sont appelés "code modificators" (`codemods` en abrégé).  Ils s'appuient sur une représentation du fichier plus riche qu'une suite de chaînes de caractères, pour que nous puissions, par exemple :
- sélectionner un import nommé sélectif CJS ;
- mais pas un import par défaut, ou un import nommé global ;
- et le remplacer par un autre import nommé, cette fois-ci en ESM, ainsi qu'une déclaration de variable.

Pour cela, ils vont effectuer un travail similaire à l'interpréteur NodeJS :
- un lexer qui transforme une chaîne de caractères en "mots" (tokens)
- un parser qui transforme cette suite de tokens en éléments du langage, organisés en arbre.

Cet arbre s'appelle un "Abstract Syntax Tree" (AST).

##### Sélectionner un élément à modifier

Voyons ce que donne le fichier contenant un import CJS dans un [constructeur d'AST](https://rajasegar.github.io/ast-builder/).

```javascript
const egg = require('./egg.js');
```
AST
```json
 {
    "type": "VariableDeclaration",
    "declarations": [
      {
        "type": "VariableDeclarator",
        "id": {
          "type": "Identifier",
          "name": "egg"
        },
        "init": {
          "type": "CallExpression",
          "callee": {
            "type": "Identifier",
            "name": "require"
          },
          "arguments": [
            {
              "type": "Literal",
              "value": "./egg.js",
              "raw": "'./egg.js'"
            }
          ]
        }
      }
    ],
    "kind": "const"
    }
```

Cet "Abstract Syntax Tree" (AST) peut être interrogé, tout comme on interroge le DOM avec Xpath: la requête s'appelle un sélecteur.

Ici, nous voulons récupérer les imports nommés globaux, le sélecteur qui nous intéresse est un nœud :
- avec affectation à une variable (sans invocation de code), donc de type `VariableDeclaration`;
- sans déstructuration, donc avec un descendant de type `Identifier`;
- d'import CJS, donc avec un descendant de type `CallExpression` et de name `require`.

Les autres syntaxes d'import (ex : import sélectif) ne sont pas inclus pour la lisibilité, mais il faut tous les comparer pour trouver leurs points communs et leurs différences, pour éviter les faux :
- positifs : sélectionner un import indésirable ;
- négatifs : ne pas sélectionner un import désiré.


##### Modifier l'élément sélectionné

Une fois le sélecteur écrit, il faut réécrire le code.
Nous avons utilisé un outil JS, [jscodeshift](https://github.com/facebook/jscodeshift), comme exemple ci-dessous.
Dans Jscodeshift, le code ne peut pas être modifié "sur place" via des mutations, il est remplacé par du code que nous allons générer. La librairie sous-jacente, [recast](https://github.com/benjamn/recast), fournit des constructeurs pour les éléments du langage.

Pour générer le code ESM suivant
```javascript
import egg from './egg.js';
```

La syntaxe est
```javascript
j.variableDeclaration("const", [j.variableDeclarator(
    j.identifier("egg"),
    j.callExpression(j.identifier("import"), [j.literal("./egg.js")])
)]);
```

Notez que certains paramètres de fonction doivent être récupérés dans l'AST précédent :
- le nom de la constante `egg`;
- le chemin du fichier `./egg.js`.

Si on assemble tous les éléments, le codemod ressemble à [ça](https://github.com/1024pix/pix/pull/5787/commits/1f5f207361cad7a04037429408b2f9545a176e4a).

#### Un périmètre de travail parfois limité

Jscodeshift suit les étapes suivantes :
- lire (récursivement) tous les fichiers d'un répertoire ;
- sur chaque fichier :
  - générer l'AST ;
  - appliquer les sélecteurs ;
  - pour chaque sélection qui matche, appeler le constructeur de code ;
  - assembler le code obtenu et réécrire le fichier d'origine.

Le plus grand périmètre de travail est le fichier, tout comme dans eslint.
Même si Jscodeshift traite les fichiers en parallèle, il n'est pas possible d'accéder à l'AST d'un autre fichier depuis l'AST courant. La conséquence : il n'est pas possible de faire une modification qui impacte deux fichiers.

Si les modifications de code ne concernent que l'intérieur d'une fonction, cela ne pose pas de problème.
Dans les cas d'ESM, qui par définition traite des interfaces entre fichiers, cela a été une sérieuse limitation.

### Se faire aider

#### Tests automatisés

Il n'est pas possible, à part avec une profonde expérience du langage et des AST, de se représenter tous les cas possibles.
Il est donc à peu près exclu de garantir que tous les cas seront traités et qu'ils le soient correctement.

Une suite de tests automatisés est donc indispensable pour :
- d'une part, documenter les cas traités ;
- détecter les cas qui ne sont pas bien traités ;
- éviter les régressions, à savoir modifier du code qui ne doit pas être modifié.

Jscodeshift embarque un framework de test dédié et les cas de test sont lisibles. Ils contiennent le code en entrée (given) et celui attendu en sortie (then).
```javascript
// input.js
const datasource = require('./datasource.js');
// output.js
import * as datasource from './datasource.js';
```

#### Lint

Après l'exécution du codemod, si l'exécution du fichier retourne une erreur, c'est que le codemod est incorrect. Si l'on inspecte le contenu du fichier, on peut constater l'erreur et corriger le codemod. Mais parfois, on a besoin d'une vue d'ensemble pour modifier le codemod pour s'assurer que tous les imports référencent un fichier existant ?

C'est là que le plugin `eslint-plugin-import` peut faire gagner du temps, par exemple en remontant tous les cas d'imports non résolus avec la règle [no-unresolved](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-unresolved.md).

Il peut aussi y avoir beaucoup de violations.

Pour y voir plus clair, vous pouvez :
- utiliser le package [eslint-nibble](https://github.com/IanVS/eslint-nibble) qui groupe par règle ;
- exécuter une seule règle sur tous les fichiers avec l'option `--rule` de `eslint` : `npx eslint --rule eslint-plugin-import/no-unresolved ./lib`
- aussi utiliser des expressions régulières avec `sed` pour extraire la liste des fichiers à modifier.

### Heuristique d'automatisation

Maintenant que vous savez qu'il est possible de modifier du code complexe, va-t-on écrire des codemods pour gérer tous les cas possibles ? Sinon, quels sont les codemods qu'il faut absolument écrire ? Voilà quelques règles, inspirées de [la matrice d'automatisation](https://xkcd.com/1205/).

##### Cas 1 - Evidence : il faut automatiser
Si une fonctionnalité CJS est reprise telle quelle dans ESM et qu'elle est utilisée dans la plupart de la codebase, écrire le codemod. Il pourrait être réalisé en script bash, mais le framework de test de jscodeshift évite les erreurs.

C'est le cas de l'import nommé.
```javascript
// CJS
const egg = require('./egg.js');
// ESM
import egg from './egg.js';
```

##### Cas 2 - Questionnement : faut-il automatiser ?
Si une fonctionnalité CJS n'est pas reprise dans ESM et est facilement simulable (ex : avec plusieurs instructions au lieu d'une), alors se demander si elle est utilisée dans la plupart de la codebase.

Si c'est le cas, écrire le codemod. Si ce n'est pas le cas, modifier les fichiers manuellement.

C'est le cas CJS de l'import + invocation dans la même instruction, remplacée par un import nommé avec assignation + invocation dans une autre instruction en ESM.

```javascript
// CJS
const ham  = new (require('./ham')).Ham();
// ESM
import { Ham } from './Ham.js';
const ham = new Ham();
```

##### Cas 3 - Impasse : il est impossible d'automatiser
Si une fonctionnalité CJS n'est pas reprise dans ESM et qu'elle n'est pas simulable facilement, la seule solution est de modifier le fichier manuellement.

C'est le cas des [exports immuables](https://2ality.com/2015/07/es6-module-exports.html) en ESM. Tous les tests CJS qui substituent une [doublure de test](https://martinfowler.com/bliki/TestDouble.html) à la volée ne sont plus valables : seule [l'injection des dépendances](https://github.com/1024pix/pix/commit/027b560b42ef72731662559f906de33ff749a008) permet de résoudre le problème. Bien sûr, il est possible en théorie d'écrire un codemod qui injecterait les dépendances : à vous de voir le coût.

## Conclusion

Nous avons vu les avantages d'ESM et qu'il était possible d'automatiser la migration du code.

Si vous envisagez une migration, le plus important est de garder l'[heuristique d'automatisation](#heuristique-dautomatisation) en tête et de vous concentrer sur le cas 2, où l'automatisation peut être envisagée, mais pas systématique. C'est ce cas qui demande le plus de recul, les deux autres cas étant évidents.

Il est en effet possible de :
- sous-estimer le coût de développement d'un codemod ;
- mais aussi de sous-estimer le caractère répétitif et frustrant de modifier des centaines de fichiers manuellement.

Nous vous conseillons de suivre votre intuition en se fixant des limites pour éviter [le biais d'investissement](https://en.wikipedia.org/wiki/Sunk_cost), puis d'apprendre en essayant. Si tout ne se déroule pas comme prévu, s'autoriser à revenir en arrière.

Par exemple, si l'on estime que :
- la modification manuelle de fichier prend 1 jour ;
- le codemod prend 2 jours ;
- et qu'au bout d'un jour de correction manuelle, on comprend qu'il reste en fait 3 jours.

Alors, on peut supprimer les modifications manuelles de fichiers et développer un codemod. Bien sûr, le développement prendra peut-être plus de temps, mais parfois les modifications manuelles peuvent saper l'énergie de l'équipe. Tout est question de contexte.

## Bonus

ESM apporte aussi les modifications suivantes :
- remplacement des pseudo-variables `__dirname` et `__filename` par la fonction `fileURLToPath` de la librairie `url` ;
- l'extension de fichier devient obligatoire dans l'import, exemple `from './egg.js'` ;
- la lecture de fichier JSON demande une clause supplémentaire `assert { type: 'json' }`, exemple `import packageJSON from '../package.json' assert { type: 'json' };`.

Voilà quelques références pour démarrer les codemods :
- [un tutoriel JScodeshift](https://medium.com/@andrew_levine/writing-your-very-first-codemod-with-jscodeshift-7a24c4ede31b) ;
- [une sandbox](https://astexplorer.net/) pour écrire des codemods (activez l'option Transform avec `recast`);
- [la définition](https://github.com/benjamn/ast-types/tree/master/src/def) des constructeurs pour générer le code.
