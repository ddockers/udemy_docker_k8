# Docker Compose with Multiple Local Containers

I want to be able to create a website that displays the number of visits to the page.

- **Node App** will be the web server to respond to http requests and display html on the page.
- **Redis** server will be used to store the data (number of times the page has been visited)

## Architecture
![Imgur](https://i.imgur.com/z9z5KsV.png)

To implement the Node app I need to compose a javascript file, as well as a json file.

### package.json
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

### Initial index.js
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

### Initial Dockerfile
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

## Docker Compose

Docker compose will be done in a YAML file. Here are the steps I need:
1. I want a container with the Redis server
2. I want it made using the *redis* image
3. I want a Node App container
4. I want it made using the Dockerfile in the current directory
5. I want to map port 8081 from the container to 4001 in my local machine

### docker-compose.yml
```
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    build: .
    ports:
      - "4001:8081"  
```

The service is a type of container. I am defining two services in this YAML file, and both these services take the form of two Docker containers.

`build: .` is telling it to look inside the pwd for a Dockerfile and use that to build this image for the container.

This docker-compose file will automatically create both containers on the same network and they'll have free access to communicate with each other.

Because I've created a Docker compose file, I can amend index.js to specify the location that the Redis server is running. I can do this by simply referring to the name of the server - *redis-server*.

The default port of a Redis server is 6379. 

### Updated index.js
```
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
    host: 'redis-server'
    port: 6379
});
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

In the visits folder, run `docker-compose up`.

![Imgur](https://i.imgur.com/RyMmLKr.png)
![Imgur](https://i.imgur.com/Ds5wi19.png)

- A network has been created so that the containers can communicate with each other
- A single instance of node-app service has been created
- A single instance of redis-server service has been created

## Troubleshooting
- `Unexpected identifier 'port'` 
Unknown what caused this issue. Amendments made:
1. docker-compose.yml changed to compose.yaml
2. Changed Redis version in package.json to 3.1.2
3. Comma added in index.js after `host: 'redis-server'`
4. Changed Redis port to 6380. Didn't work, changed back to 6379.
5. Ran again using `docker-compose up --build`

![Imgur](https://i.imgur.com/lk7zkdG.png)

Now if I go to localhost:4001 my app is up and running.

![Imgur](https://i.imgur.com/hWf2wib.png)

Hitting *Refresh* increases the visit count.