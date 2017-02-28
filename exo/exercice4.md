# Exercice 4 - Map as Object - Object as Map, Symbol, Unit Tests et propertyDescriptor

Take a deep breath and dig deep ;-)

------------

Sur une idée de [Fraxken](https://github.com/fraxken).

Conçu et rédigé par [Purexo](https://github.com/Purexo).

## Description
Vous est fourni un code à completer.

J'ai voulu cet exercice complet et accessible à différents niveaux (via plusieurs paliers).  
Dans cet exercice vous allez pouvoir développer progressivement tout en étant guidé par des Tests Unitaires.

L'idée générale à travers cet exercice est de trouver un moyen pour que ce qui s'applique à un objet se répercute sur un autre et vice versa, tout en replongeant dans certaines bases méconnues du JS.

Vous aurez réussi l'exercice si tous les asserts passent (une ligne apparaitra pour vous féliciter dans votre console).

## Contraintes
Ce sont les contraintes habituelles à nos exercices. c'est à dire :

- du pur JS (pas de lib externe donc)
- la dernière version de Node pour être tranquille
- vous pouvez le faire dans un navigateur mais ce sera moins pratique ;-)

## Le code (à completer donc)

```js
const assert = require('assert');

const bar = Symbol('bar');
const map = new Map([
  ['foo', 'bar'],
  [bar, 'baz'],
  [Symbol.for('bar'), 'foo']
]);

/*
 * TODO
 * Votre travail commence ici
 */
const something = null;

/* Et ce termine la. */

/* --- Place aux tests --- */
// Difficulté 1
// Vérification que something est iso par rapport à la map
assert.doesNotThrow(
  () => assert.equal(something.foo, 'bar'),
  TypeError,
  `Vous avez lancé l'exercice sans coder pour voir ce que ça faisait ?
c'est bien c'est un bon reflexe de TDD, maintenant que vous savez que ça ne fonctionne pas, vous pouvez réfléchir à une solution.
Pensez bien à lire les tests et les commentaires pour voir ce qui est attendu ;-)`
);
assert.equal(something.foo, 'bar');
assert.equal(something[bar], 'baz');
assert.equal(something[Symbol.for('bar')], 'foo');

// Difficulté 1.5
assert.equal(Object.getOwnPropertyNames(something).length, 1); // les clés en string
assert.equal([...Object.keys(something)].length, 1); // les clés énumérables en string
assert.equal(Object.getOwnPropertySymbols(something).length, 2); // les clés en Symbol
assert.equal(Reflect.ownKeys(something).length, 3); // toutes les clés

// Difficulté 2
// Ce que l'on applique à something ce répercute dans map
something[bar] = bar;
assert.equal(map.get(bar), bar);

something['new property'] = 'Oh Yeah';
assert(map.has('new property'));
assert.equal(map.get('new property'), 'Oh Yeah');

// Dificulté 2.5
// Et Vice Versa
map.set('property new', 'hello');
assert('property new' in something);
assert.equal(something['property new'], 'hello');

// Difficulté 3
// operateur delete
delete something['property new'];
assert.equal(map.has('property new'), false);

// et dans l'autre sens
map.delete('new property');
assert.equal('new property' in something, false);

/*
 * Exercice Terminé
 */
console.log(`Félicitations, vous êtes arrivé au bout du script sans que les asserts ne lèvent d'exception`);

// Bonus Point
// Si vous êtes arrivés au bout, ce code devrait fonctionner sans souci
// ça permet de s'assurer que la map et something sont iso sous tous points de vues en termes de données stockées
// à décommenter si vous souhaitez vérifier

/*
const duplicatedMapFromSomething = new Map(Reflect.ownKeys(something).map((key => [key, something[key]])));
assert.equal(duplicatedMapFromSomething.size, map.size);
for (let [key, value] of duplicatedMapFromSomething) {
  assert.equal(value, map.get(key));
}
*/
