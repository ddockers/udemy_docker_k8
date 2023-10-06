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

`mkdir` multi_cont_app -> complex -> worker

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