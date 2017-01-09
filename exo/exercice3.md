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

# Server code 

```js
const net = require('net');

const userValidation = new RegExp(/^[a-zA-z0-9]{3,10}$/);
const messageRegex = new RegExp(/^(\/auth|\/msg|\/logout|\/exit)\s*(.*)$/);

const registryName = {};
const clients = [];
const messagesHistory = [];

const server = net.createServer(function(socket) {

    clients.push(socket);
    socket.setTimeout(360000,function() {
        socket.end('Socket disconnect - Time out\r\n');
    });

    console.log('socket connected with remoteAddr => '+socket.remoteAddress+' and remote port => '+socket.remotePort);
    if(messagesHistory.length > 0) {
        messagesHistory.forEach( Buf => {
            socket.write(`${Buf}\r\n`);
        });
    }

    socket.protect = false;
    socket.on('data',Buf => {
        try {
            const str = Buf.toString();
            let data = str.match(messageRegex);
            if(data) {
                socket.emit('command',data[1],data[2]);
            }
            else {
                socket.write(Buffer.from('Invalid message format!\r\n'));
            }
        }
        catch(err) {
            console.log('Failed to parse Buffer!');
            // Silent
        }
    });

    socket.on('error', err=> {
        // Silent
    });

    socket.on('message',msg => {
        console.log(`Message triggered by ${socket.username}`);
        const finalMsg = Buffer.from(`${socket.username}: ${msg}\r\n`);
        addHistory(finalMsg);
        broadcast(finalMsg,socket);
    });
    
    socket.on('command',(command,value) => {
        if(command === '/auth') {
            if(socket.username !== undefined) {
                socket.write(Buffer.from(`Already logged in as ${socket.username}\r\n`));
            }
            else {
                if(userValidation.test(value)) {
                    if(registryName.hasOwnProperty(value)) {
                        socket.write('This name is already taken!\r\n');
                    }
                    else {
                        registryName[value] = true;
                        socket.username = value;
                        const finalMsg = Buffer.from(`${socket.username}: is now connected to the tchat!\r\n`);
                        addHistory(finalMsg);
                        broadcast(finalMsg,socket);
                    }
                }
                else {
                    socket.write('Invalid username format!');
                }
            }
        }
        else if(command === '/msg') {
            if(socket.username !== undefined) {
                if(socket.protect) {
                    socket.write('Dont spam the tchat idiot!');
                }
                else {
                    if(value.trim() !== '') {
                        socket.protect = true;
                        socket.emit('message',value);
                        setTimeout(() => {
                            socket.protect = false;
                        },100);
                    }
                }
            }
            else {
                socket.write(Buffer.from('You have to be authenticated before posting any messages!\r\n'));
            }
        }
        else if(command === '/logout' && socket.username !== undefined) {
            const finalMsg = Buffer.from(`${socket.username}: is now disconnected\r\n`);
            addHistory(finalMsg);
            broadcast(finalMsg,socket);
            delete registryName[socket.username];
            socket.username = undefined;
        }
        else if(command === '/exit') {
            if(socket.username) {
                console.log(`Authentified user => ${socket.username} is now disconnect`);
                const finalMsg = Buffer.from(`${socket.username}: is now disconnected\r\n`);
                addHistory(finalMsg);
                broadcast(finalMsg,socket);
                delete registryName[socket.username];
            }
            socket.end('Exit triggered - Good bye !');
        }
        else {
            socket.write(Buffer.from(`Invalid command ${command}`));
        }
    });

    socket.on('close',_ => {
        if(socket.username) {
            console.log(`Authentified user => ${socket.username} is now disconnect`);
            const finalMsg = Buffer.from(`${socket.username}: is now disconnected\r\n`);
            addHistory(finalMsg);
            broadcast(finalMsg,socket);
            delete registryName[socket.username];
        }
        else {
            console.log(`Socket for ${socket.remoteAddress}:${socket.remotePort} disconnect`);
        }
        clients.splice(clients.indexOf(socket), 1);
    });

    function addHistory(msg) {
        messagesHistory.push(msg)
        if(messagesHistory.length > 20) {
            messagesHistory.shift();
        }
    }

    // Send a message to all clients
    function broadcast(message, sender) {
        clients.forEach( client => {
            client.write(message);
        });
    }
});

server.listen(3000);
console.log('Server started on port 3000');
```
