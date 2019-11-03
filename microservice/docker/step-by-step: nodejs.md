# Set up Node-js

This is a tutorial about setting a node-js environment with docker.

## create app.js

```
vim app.js
```

```js
const http = require('http');
const os = require('os');
console.log("Kubia server starting...");
var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);
```

## create Dockerfile
```
vim Dockerfile
```

```
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

## Build Image
```
docker build -t kubia .
```

## Run and check
```
docker run --name kubia-container -p 8080:8080 -d kubia
```

```
curl localhost:8080
```
or visit ```(ip-address):8080```

## Push to DockerHub
```
docker login
```

```
docker push luksa/kubia
```

