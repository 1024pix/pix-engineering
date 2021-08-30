---
layout: post
title:  "Modifier une clef primaire"
date:   2021-08-30 10:10:42 +0200
categories: bdd
excerpt: 
---

Friends don't let friends use INTEGER.
Le propos est peut etre un peu appuyé, mais il exprime bien la réalité.
Il est possible de modifier le type d'une colonne, mais cela a un cout lorsque la table est remplie. 
Dans le cas d'une application qui ne pouvait pas deviner quel sera le succès, et en présence du choix d'entier comme identifiant
(ajouter lien vers utilisation des uuid), le gain d'espace peut être négligeable par rapport à une interruption de service, même planifiée.

## TODO:
Ajouter volumétrie et sensiblité  à la seconde si grandit

S'il faut modifier tous les enregistrements d'une table, nous sommes confiants.
Mais si les relatiosn sont linéaires, une opération instantanée (ex: 10 ms) peut se transformer en cauchemar (ex: 10 ms * 1000 = > durée migratiopn post hookk)

Mentionner l'article d'origien (coffee & bagekls) et post SO

Ajouter que chnager type (INTEGER => BIG INTEGER) entraien coercition de type.


## Long story short


// On part de là
CREATE TABLE foo (ID INTEGER PRIMARY KEY);
INSERT INTO foo generate_series(1, 1e6);


// Habituel
ALTER TABLE foo COLUMN idBigInteger BIG INTEGER; <= toutes les autres transactions sont bloquées, plusieurs heures => downtime


// Notre choix en pseudo-SQL
ALTER TABLE foo ADD COLUMN idBigInteger NOT NULL DEFAULT -1;
// Traiter les tuples existants
UPDATE foo SET idBigInteger = id;

// Couper les accès

// Traiter les tuples créés depuis la création de la colonne
UPDATE foo SET idBigInteger = id;
CREATE UNIQUE INDEX indexBigIntId;
ALTER TABLE foo DROP COLUMN id;
ALTER TABLE foo RENAME COLUMN bigIntId TO id;
ALTER TABLE foo ADD PRIMARY KEY id USING indexBigIntId;


La clef est d'effectuer la conversion des données en amont.

Ce scénario est incomplet:
- le premier UPDATE est très long et consomme beaucoup de ressources;
- l'accès à la BDD doit être arrêté pendant une période

Pour savoir plus, voir le chapitre XX





## Qu'est-ce qu'une clef primaire ?

Nous pourrions oublier que la ligne suivante
CREATE TABLE foo (id SERIAL PRIMARY KEY);

Est équivalente à
```sql
CREATE TABLE foo ( id INTEGER );
CREATE SEQUENCE seq_id TYPE INTEGER;
ALTER TABLE foo COLUMN id SET DEFAULT seq_id.nextval();
ALTER TABLE foo ADD CONSTRAINT UNIQUE(id);
ALTER TABLE foo ADD CONSTRAINT NOT NULL(id);
```

Le fait que le mot-clef PRIMARY KEY ne soit pas mentionné
a en réalité un impact: il est possible d'ajouter plusieurs
contraintes NOT NULL UNIQUE à un ensemble de colonnes, alors qu'il
n'y a qu'une seule clef primaire par table. Cette notion est
avant tout sémantique, elle indique comme identifier une ligne.
Pour rappel, il est aussi possinble de séparer l'identifiant technique et fonctionnel.
Et se demander qui doit effectuer le contrôle. (UNIQUE sur plusieurs colonnes à l'insertion respecté => un ID technique).

## Qu'est-ce qu'un modification de colonne ?


Le MVCC de Postgresql peut se résumer ainsi:
Si
- une transaction A démarre
- une transaction B, ayant démarré après A, modifie un enregistrement, puis finit avec succès
- une transaction C démarre après la fin de B
  Alors, pour qu'en même temps
- la transactin A voit l'ancienne version
- la transaction C voit la nouvelle version
  Il faut que ces 2 versions soient stockées à des endroits différents (même si l'on parle du même enregistrement).

Un UPDATE est en réalité
- un INSERT d'une nouvelle version
- suivi d'un DELETE de l'ancienne version quand les transactions qui y avaient accès ont terminées.

Il n'y a d'ailleurs pas à proprement parler de DELETE (voir visibility map), mais ce n'est pas le propos ici.

La modification du type d'une colonne est en réalité quelque chose comme :
- ALTER TABLE foo ADD COLUMN bar_new_type
- INSERT INTO foo (bar_new_type) SELECT to_new_type(f.bar) AS foo FROM foo;
- ALTER TABLE foo DROP COLUMN bar;
- ALTER TABLE foo RENAME COLUMN bar_new_type TO bar.

Le point critique ici est l'UPDATE qui va créer, à partir de N enregistrements, N' autres enregistrements.
Si d'autres transactions sont encore en cours, il faut garder 2N enregistrements (doubler la taille de la table) tant qu'il existe encore au moins une transaction.


Pour cette raison et probablement d'autres, PostgreSQL évite ce scénario en attendant que toutes les transactions
soient achevées avant d'exécuter le changement: il demande un verrou en accès exclusif. Après ce changement,
il y toujours 2N enregistrements tant que la transaction n'a pas été validée (COMMIT). On pourrait se dire que ces données
sont inutiles, mais pour annuler rapidement la transaction, il faut les conserver.



## Comment permettre les accès concurrents

Pour que les données puissent continuer à être insérées entre la création de la colonne et sa promotion effective,
nous avons utilisé des trigger.


## Anticiper et lisser la charge ?

Charge:
- Alimenter la nouvelle colonne
  - sur les tuples existants, sur des ensembles (batch)
    - sur les nouveaux tuples, par trigger
- Valider la contrainte NOT NULL
  - sur les tuples existants: en une fois, en fournissant une valeur par défaut, facilement identifiable
  - sur les nouveaux tuples, au fil de l'eau par trigger
- Valider la contrainte d'unicité
  - sur l'ensemble des tuples, sans bloquer les accès concurrents, par création d'index concurrente

Python syntaxe a<=>b ou jeu de carte


batc: stratégie évident n'est pas toujours la meilleure
- permettre un traitement dans la BDD uniquement + ne pas forcer d'ordre de parcours + gère l'interrutpion (stateless, auto-supporté)
- nosu avoons choisi une solution hors de la BDD en forçant l'ordre de parcours + pas de gestion interruption (stateful, externe)

Au vu de l'impact (acceptable en temps calme), nous allons complexifier le traitement (Node + table BDD) pour permettre
- de réduire son impact en temps réel (raccourcir la boucle de feedback)
- la reprise après interruption

## Assurer la cohérences des données lors de l'échanges des colonnes


PostgreSQL détermine, pour chaque requête, le verrou approprié.
C'est pour cela que la version "1 ligne" est sécurisée:
- elle ne corrompra pas de données
- elle aboutira

Mais pour nous, ce n'est pas le cas: des insertions échouent si elles ont lieu dans l'intervalle où id n'existe plus.
Il est nécessaire de mettre en file d'attente ces insertions, et nous pouvons le faire en demandant explicitement un verrou exclusiof sur la table avec LOCK TABLE foo ACCESS EXCLUSIVE.
Si cette période d'échange ne dure qu'in instant, le traffic n'est pas interrmpu avec l'exteieur et n'est pas remaquré.


## Si on recolle les bouts, ça donne
En pseudo SQL / pgPLSQL / nodeJS

// Instantané
ALTER TABLE foo ADD COLUMN idBigInteger NOT NULL DEFAULT -1;
CREATE TRIGGER WHEN INSERT INTO foo FOR EACH ROW :NEW.bigIntId = :new.id;

// Traitement de migration en nodeJS - 48h
SELECT MAX(id) INTO maxId FROM foo;
for (const i = 0, i <= maxId, i++){
UPDATE foo SET idBigInteger = id WHERE id BETWEEN i AND (i + chunkSize -1);
}
CREATE UNIQUE INDEX CONCURRENTLY indexidBigInteger;

// Quand le traitement est fini - Instantané
BEGIN TRANSACTION;
LOCK foo WITH ACCESS EXCLUSIVE;
ALTER SEQUENCE seqPkFoo TYPE BIG INTEGER;
ALTER TABLE foo ALTER COLUM bigIntId SET DEFAULT seqPkFoo.nextval();
ALTER TABLE foo DROP COLUMN id;
ALTER TABLE foo RENAME COLUMN idBigInteger TO id;
ALTER TABLE foo ADD PRIMARY KEY id USING indexidBigInteger;
COMMIT TRANSACTION;

A partir de là, la création de la PK est une simple opération sur les méta-données de la table.
Pendant ce temps, toutes les autres transactions sont bloquées, mais cela pendant moins d'une seconde => "No" downtime
