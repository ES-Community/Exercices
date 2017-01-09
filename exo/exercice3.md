# Net tchat

Dans cet exercice il va vous falloir vous connecter à un serveur TCP NodeJS en tant que client. Voiçi la liste des commandes à votre disposition : 

- /auth {username}
- /msg {message}
- /logout
- /exit 

Voici les regex utilisé en back-end : 

```js
const userValidation = new RegExp(/^[a-zA-z0-9]{3,10}$/);
const messageRegex = new RegExp(/^(\/auth|\/msg|\/logout|\/exit)\s*(.*)$/);
```

> Attention le serveur vous renverra un historique des messages lors de la connexion !

## Contraintes 

- Utiliser uniquement les packages NodeJS pour vous connecter et tchater avec les autres.

## Code

Voici un code de base pour vous aider : 

```js
const net = require('net');
const readline = require('readline');

const client = net.connect({port: 3000}, () => {
    console.log('connected to server!');
});
```

Le serveur distant pour l'exercice est `78.213.166.192:3000`
