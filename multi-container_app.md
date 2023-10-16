# Building a Multi-Container App

I'm going to be building an app where a user will enter the index of the Fibinacci sequence and the valuse of the index will be returned.

![Imgur](https://i.imgur.com/zXRD06R.png)

### Architecture
![Imgur](https://i.imgur.com/UJHsOD8.png)

If the browser is tring to access frontend data, like a JS file, the traffic will be routed through the React server.

If the browser wants to access a backend API (to retrieve values), the traffic is routed through the Express server.

![Imgur](https://i.imgur.com/1U6z5qe.png)

*Values I have seen* are stored in a Postgres db.

*Calculated values* are stored in a separate Redis db.

### App Flow

![Imgur](https://i.imgur.com/TT8qcBL.png)

1. Index is entered
2. React App makes API request to backend Express server
3. Express server stores index in Postgres db
4. Express server puts index into Redis db
5. When a new number shows up in Redis db, the Worker will run a Nodejs process
6. Worker calculates Fibonacci value and sends it back to Redis
7. React App requests the number from Redis and it is shown on the screen.

## Setup

1. `mkdir` multi_cont_app -> complex -> worker

In the worker directory, add the following:

`package.json`

```
{
  "dependencies": {
    "nodemon": "1.18.3",
    "redis": "2.8.0"
  },
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon"
  }
}
```

`index.js`

```
const keys = require('./keys');
const redis = require('redis');

const redisClient = redis.createClient({
  host: keys.redisHost,
  port: keys.redisPort,
  retry_strategy: () => 1000,
});
const sub = redisClient.duplicate();

function fib(index) {
  if (index < 2) return 1;
  return fib(index - 1) + fib(index - 2);
}

sub.on('message', (channel, message) => {
  redisClient.hset('values', message, fib(parseInt(message)));
});
sub.subscribe('insert');
```

`keys.js`

```
module.exports = {
  redisHost: process.env.REDIS_HOST,
  redisPort: process.env.REDIS_PORT,
};
```

2. `mkdir` multi_cont_app -> complex -> server

`package.json`

```
{
  "dependencies": {
    "express": "4.16.3",
    "pg": "8.0.3",
    "redis": "2.8.0",
    "cors": "2.8.4",
    "nodemon": "1.18.3",
    "body-parser": "*"
  },
  "scripts": {
    "dev": "nodemon",
    "start": "node index.js"
  }
}
```

`keys.js`

```
module.exports = {
  redisHost: process.env.REDIS_HOST,
  redisPort: process.env.REDIS_PORT,
  pgUser: process.env.PGUSER,
  pgHost: process.env.PGHOST,
  pgDatabase: process.env.PGDATABASE,
  pgPassword: process.env.PGPASSWORD,
  pgPort: process.env.PGPORT,
};
```

`index.js`

```
const keys = require('./keys');

// Express App Setup
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Postgres Client Setup
const { Pool } = require('pg');
const pgClient = new Pool({
  user: keys.pgUser,
  host: keys.pgHost,
  database: keys.pgDatabase,
  password: keys.pgPassword,
  port: keys.pgPort,
  ssl: { rejectUnauthorized: false },
});

pgClient.on('connect', (client) => {
  client
    .query('CREATE TABLE IF NOT EXISTS values (number INT)')
    .catch((err) => console.error(err));
});

// Redis Client Setup
const redis = require('redis');
const redisClient = redis.createClient({
  host: keys.redisHost,
  port: keys.redisPort,
  retry_strategy: () => 1000,
});
const redisPublisher = redisClient.duplicate();

// Express route handlers

app.get('/', (req, res) => {
  res.send('Hi');
});

app.get('/values/all', async (req, res) => {
  const values = await pgClient.query('SELECT * from values');

  res.send(values.rows);
});

app.get('/values/current', async (req, res) => {
  redisClient.hgetall('values', (err, values) => {
    res.send(values);
  });
});

app.post('/values', async (req, res) => {
  const index = req.body.index;

  if (parseInt(index) > 40) {
    return res.status(422).send('Index too high');
  }

  redisClient.hset('values', index, 'Nothing yet!');
  redisPublisher.publish('insert', index);
  pgClient.query('INSERT INTO values(number) VALUES($1)', [index]);

  res.send({ working: true });
});

app.listen(5000, (err) => {
  console.log('Listening');
});

```

## Dockerising the React App

The React App code has been downloaded and saved in `complex -> client`. Inside `client` is where `src/` is located.

Dev Dockerfiles need to be made for the below.

![Imgur](https://i.imgur.com/zM2rXaG.png)

If changes are made to the code, we need to make sure it works before pushing to production.

![Imgur](https://i.imgur.com/2sOlpRN.png)

The fact that the Docker compose shares files means that if a change is made to the source code, the image does not need to be built again from scratch.

### Client Dockerfile.dev
Inside `client` create the below Dockerfile.dev:

```
FROM node:16-alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

`cd` into `client` and run:

```
docker build -f Dockerfile.dev .
```

Run the created container using `docker run <container ID>`.

### Server Dockerfile.dev

Inside `server` create the following Dockerfile.dev:

```
FROM node:14.14.0-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

### Worker Dockerfile.dev

Same as above. 

## Docker Compose

Refer back to architecture:

![Imgur](https://i.imgur.com/UJHsOD8.png)

Requirements:
- React server needs to be available on a given port
- Express server needs to be available on a given port
- Worker needs to be able to connect to Redis
- The correct environment variables for Redis and Postgress need to be provided to the express server and the worker

![Imgur](https://i.imgur.com/8fP1v9p.png)

Specify build - what Dockerfile to use?
Specify volumes - if the source code is changed the container code needs to update as well
Specify env variables - env variables can be seen in server -> keys.js

```
module.exports = {
  redisHost: process.env.REDIS_HOST,
  redisPort: process.env.REDIS_PORT,
  pgUser: process.env.PGUSER,
  pgHost: process.env.PGHOST,
  pgDatabase: process.env.PGDATABASE,
  pgPassword: process.env.PGPASSWORD,
  pgPort: process.env.PGPORT,
};
```

The server relies on the env variables to decide how to connect to instance of Redis and Postgres server.

Inside `complex` create `docker-compose.yml`.

### Nginx
Nginx is used to route to the correct server. 

Nginx redirects API requests to the express server and other requests to the react server.

![Imgur](https://i.imgur.com/joPHMzq.png)

A folder called `nginx` needs to be added to `complex`. Here is where the nginx configuration will be.

![Imgur](https://i.imgur.com/CNldHjU.png)

### docker-compose.yml

```
version: '3'
services:
  postgres:
    image: 'postgres:latest'
  redis:
    image: 'redis:latest'
  nginx:
    restart: always
    build:
      dockerfile: Dockerfile.dev
      context: ./nginx
    ports:
      - '3050:80'
  api:
    build:
      dockerfile: Dockerfile.dev
      context: ./server
    volumes:
      - /app/node_modules
      - ./server:app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PGUSER=postgres
      - PGHOST=postgres
      - PGDATABASE=postgres
      - PGPASSWORD=postgres_password
      - PGPORT=5432
  client:
    build:
      dockerfile: Dockerfile.dev
      context: ./client
    volumes: 
      - /app/node_modules
      - ./client:app
  worker:
    build:
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker:app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
```