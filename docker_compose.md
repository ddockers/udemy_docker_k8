# Docker Compose with Multiple Local Containers

I want to be able to create a website that displays the number of visits to the page.

- **Node App** will be the web server to respond to http requests and display html on the page.
- **Redis** server will be used to store the data (number of times the page has been visited)

## Architecture
![Imgur](https://i.imgur.com/z9z5KsV.png)

To implement the Node app I need to compose a javascript file, as well as a json file.

## package.json
```
{
    "dependencies": {
        "express" : "*",
        "redis" : "2.8.0"
    },
    "scripts" : {
        "start" : "node index.js"
    }
}
```

## index.js
```
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient();
client.set('visits', 0);

app.get('/', (req, res) => {
    client.get('visits', (err, visits) => {
        res.send('Number of visits is ' + visits);
        client.set('visits', parseInt(visits) + 1);
    });
});

app.listen(8081, () => {
    console.log('Listening on port 8081');
});
```

## Initial Dockerfile
```
FROM node:alpine

WORKDIR '/app'

COPY package.json .
RUN npm install
COPY . .

CMD ["npm", "start"]
```

I have used `mkdir visits` to create a directory in which to run the app.

I can `cd` into the folder and run:
```
docker build .
```
This creates an image of the Docker file and runs the Docker build in the pwd.

I can tag it using `docker build -t ddoxton/visits: latest .`.

When I run it using `docker run ddoxton/visits` I get the following error:

![Imgur](https://i.imgur.com/kvBeErT.png)

This is because it cannot connect to the Redis server as it is not running.

I can run `docker run redis` to start up another container with redis running.

When I open a new terminal to run `docker run ddoxton/visits` I still receive the same error.

This is because the Node App container and the Redis container don't automatically have a connection between them.

Networking infrastructure needs to be set up between them. The easiest way to do this is using **Docker Compose**.