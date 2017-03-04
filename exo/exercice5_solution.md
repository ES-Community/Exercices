# Exercice 5 - Bus

Pour cet exercice vous pouviez le faire de plusieurs manières. Nottament avec des streams et des events (ou bien que avec des events). Pour ma
part j'ai choisi une structure entière composé de streams en in/out.

```js
'use strict';
const stream = require('stream');
const events = require('events');

class Node extends stream.Duplex {
    constructor() {
        super();
    }
    _read() {}
    _write() {}
}

class GatewayNode extends stream.Transform {
    constructor() {
        super();
    }
    _read() {}
    _write(chunk,_,next) {
        this.push(chunk);
        next();
    }
}

/* 
    Network Package 
*/
class NetworkPackage extends events {
    constructor(header,content) {
        super();
        this.header     = header;
        this.content    = content;
        this.id         = NetworkPackage._generateID(10);
        this.expected_out        = 1;
        this._in_nb              = 0;
        this._out_nb             = 0;
        NetworkPackage.map[this.id] = this;
        this._eTimeout = setTimeout(() => {
            this.emit('timeout');
            delete NetworkPackage.map[this.id];
        }, NetworkPackage.defaultTimeout || 5000 );
    }
    _close() {
        clearTimeout(this._eTimeout);
        delete NetworkPackage.map[this.id];
        this.emit('done');
    }
    _in_receive() {
        this._in_nb++;
        return this._in_nb === this.expected_in;
    }
    _out_receive() {
        this._out_nb++;
        return this._out_nb === this.expected_out;
    }
}

NetworkPackage.defaultTimeout = 5000;
NetworkPackage._generateID = function(N) {
    return (Math.random().toString(36)+'00000000000000000').slice(2, N+2);
}
NetworkPackage.map = {};

/*
    Network Node
*/
class NetworkNode {
    constructor(name,gateway) {
        this.name           = name;
        this.gateway        = gateway;
        this.linkedNodes    = [];
        this.activeMode( gateway === true ? 'gateway' : 'normal');
    }
    activeMode(mode) {
        if(mode === 'gateway') {
            this.in     = new GatewayNode();
            this.out    = new GatewayNode();
        }
        else {
            this.in     = new Node();
            this.out    = new Node();
        
            this.in._write = (chunk,_,next) => {
                const networkPKG = NetworkPackage.map[chunk.toString()];
                if(networkPKG.header.to !== this.name) {
                    this.in.push(chunk);
                    next();
                    return;
                }
                
                if( networkPKG._in_receive() ) {
                    networkPKG.rc_code = 200;
                    networkPKG.body = 'OK';
                    if (networkPKG.header.back === true) {
                        this.out.write(networkPKG.id);
                    }
                    else {
                        networkPKG._close();
                    }
                }
                next();
            };
            this.out._write = (chunk,enc,next) => {
                const networkPKG = NetworkPackage.map[chunk.toString()];
                if(networkPKG.header.from !== this.name) {
                    this.out.push(chunk) && next();
                    return;
                }
                networkPKG._out_receive() && networkPKG._close();
                next();
            };
        }
    }
    forward(opts) {
        if(this.gateway === false) return;
        const networkPKG = new NetworkPackage({
            from: this.name,
            to: opts.to,
            back: opts.back || false
        },opts.content);
        networkPKG.expected_in = this.linkedNodes.length;
        process.nextTick(() => {
            if( this.in.write(networkPKG.id) === false ) {
                this.in.once('drain',() => {
                    this.in.write(networkPKG.id);
                });
            }
        });
        return networkPKG;
    }
    pipe(node) {
        if(node instanceof NetworkNode) {
            this.in.pipe(node.in);
            node.out.pipe(this.out);
            this.linkedNodes.push(node.name);
        }
        else {
            throw new Error('Invalid piping');
        }
    }
}

module.exports = {
    Node,
    GatewayNode,
    NetworkPackage,
    NetworkNode
};

const Benchmark = require('benchmark');

const a = new NetworkNode('a');
const b = new NetworkNode('b',true);
const c = new NetworkNode('c');
const d = new NetworkNode('d',true);
const e = new NetworkNode('e');

a.pipe(b);
b.pipe(c);
b.pipe(d);
d.pipe(e);

let suite = new Benchmark.Suite;
 
// add tests 
suite.add('forward#Event', function() {
    a.forward({
        to: "e",
        content: "hello world!"
    });
})
.on('cycle', function(event) {
  console.log(String(event.target));
})
.on('complete', function() {
  console.log('done');
})
.run({ 'async': true });
```

A noter que j'ai utiliser le package benchmark pour faire un benckmark de vitesse d'exécution plutôt classique (n'exprime pas la teneur en charge néanmoins).
