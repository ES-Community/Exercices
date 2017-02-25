# Exercice 2 - HTTP Serving

## Introduction

Vous avez les bases d'un serveur http. L'objectif est de passer au client le contenu des fichiers se trouvant dans un dossier qui se nomnera `static`.

Par exemple vous avez un fichier **test.txt** dans le répertoire **static** avec à l'intérieur la chaine string `Hello world`. 
Lorsque que le client se connecte sur `localhost:3000/test.txt` la page affichera `hello world` ou le message d'erreur (identique à ceux présent dans la section output solution).

## Projet - les fichiers

```
server.js
static
    text.txt (hello world) 
```

Rien ne vous interdit de faire vos propres fichiers (tant que le principe est le même).

## Contraintes

- Seul les packages du core NodeJS sont autorisés.
- Nous n'attendons aucun système de cache.
- L'utilisation de code Synchrone est interdite.

## Code 

Vous êtes libre de le faire en TypeScript ou Babel. L'utilisation du flag **--harmony** est autorisée.

- Vous êtes libre de modifier le port du serveur http.

##### ES6 Version : 
```js
const http = require('http');

const server = http.createServer( (request, response) => {
    const method    = request.method;
    const url       = request.url; 

    console.log(`method: ${method}`);
    console.log(`url: ${url}`);

    // Continue the code here ! 
});

server.listen(3000);
```

##### TypeScript Version : 
```ts
import * as http from "http";

const server : http.Server = http.createServer( (request: http.IncomingMessage, response: http.ServerResponse) => {
    const method: string    = request.method;
    const url: string       = request.url; 

    console.log(`method: ${method}`);
    console.log(`url: ${url}`);

    // Continue the code here ! 
});

server.listen(3000);
```

## Output solution

```
method: GET
url: /test.txt
Hello world
method: GET
url: /favicon.ico
The file does not exist!
method: GET
url: /urltest
Error 404 - This URL is not a file!
method: GET
url: /mdr.txt
The file does not exist!
```
