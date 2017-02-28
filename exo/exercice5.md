# Bus

L'objectif de cet exercice est la création d'un Bus de messages (ou d'un réseau de noeuds). Une fois le réseau construit les messages y naviguent automatiquement.

La structure de notre exercice sera celle-ci :

```
A<-->B<-->C
     ^
     | 
     v
F<-->D<-->E
```

L'objectif est d'envoyer un message sur le point 'A' vers le point 'E' par exemple. Une fois le message arrivé au point E il repartira sur le point A (si souhaité).

# Contrainte

Seul les packages du core NodeJS suivant sont autorisés : 

- Events
- Streams

# Objectifs 

- A. Faire naviguer un message du point 'A' au point 'E' ou 'F'
- B. Retourner une réponse du point 'E' ou 'F' vers le point 'A' (dans la continuiter A).
- C. Etre capable de connaître l'état du message à tout moment.
- D. Etre capable de supporté plusieurs branches. (Modification du schéma de base nécessaire)


```
A<-->B<-->C
^    ^
|    | 
v    v
D----D<-->E
```

**bonus:**

- E. Optimisation des chemins les plus cours.

# Code

Code de fin de ma version perso : 

```js
const a = new NetworkNode('a');
const b = new NetworkNode('b',true);
const c = new NetworkNode('c');
const d = new NetworkNode('d',true);
const e = new NetworkNode('e');

a.pipe(b);
b.pipe(c);
a.pipe(d);
b.pipe(d);
d.pipe(e);

console.time('routing');
const NPKG = a.forward({
    to: "e",
    content: "hello world!"
});
NPKG.on('done', _ => {
    console.timeEnd('routing');
    console.log('done');
    console.log(NPKG.body);
});
NPKG.on('timeout', _ => {
    console.log('timeout');
    NPKG = undefined;
});
```
