# Serving solution 

Multiple solution can be imagined. The url matching is not really perfect (depending on what you expect from the routing / serving). 
I personnaly use stat to complete by http request header (i think it's the best methods).

The goal of this exercice is to use fs.createReadStream instead of fs.readFile.

```js
const http = require('http');
const path = require('path');
const fs   = require('fs');

const mimeTypes = {
    "html": "text/html",
    "jpeg": "image/jpeg",
    "jpg": "image/jpeg",
    "png": "image/png",
    "js": "text/javascript",
    "css": "text/css",
    "txt": "text/plain",
    "json": "application/json"
};

const server = http.createServer( (request, response) => {
    const method    = request.method;
    const url       = request.url; 

    console.log(`method: ${method}`);
    console.log(`url: ${url}`);

    const url_extension = path.extname(url);
    if(url_extension !== '' || url.includes('.')) { // Check extension equal nothing (can be improved).
        const filePath = path.join( __dirname , 'static' , url ); // Resolve ADDR
        console.log(`filePath => ${filePath}`);
        fs.stat( filePath , (err,stats) => { // Check if the file exist
            if(err || !stats.isFile()) {
                console.log('The file does not exist!');
                response.statusCode = 404;
                response.end('The file does not exist!');
                return;
            }

            const mimeType = mimeTypes[path.extname(filePath).split(".").pop()];
            const RStream = fs.createReadStream( filePath );

            RStream.on('open',_=>{
                response.writeHead(200,{
                    'Content-Type': mimeType,
                    'Content-Length': stats.size
                });
                RStream.pipe(process.stdout); // Pipe to the terminal output
                RStream.pipe(response); // Pipe to the response.
            });

            RStream.on('error',err=> { // Catch error
                console.log('Internal server error');
                response.statusCode = 500;
                response.end('Internal server error');
            });

            RStream.on('end',_=> { // Close response on end.
                response.end();
            });
        });
    }
    else {
        /*
            In normal case you handle Routing here!
        */
        console.log('Error 404 - This URL is not a file!');
        response.statusCode = 404;
        response.end('Error 404 - This URL is not a file!');
    }
});

server.listen(3000);
```
