# Deployment of Multi-Container App

## Continuous Integration Workflow

This is how the app will be deployed.

![Imgur](https://i.imgur.com/LVV7mWH.png)

### Development vs Production Architecture

![Imgur](https://i.imgur.com/UJHsOD8.png)

![Imgur](https://i.imgur.com/Kr7o1UF.png)

In production, Dockerfile.dev files were used in *worker, client* and *server*.

Now, Dockerfiles need to be created for production.

### Server Dockerfile

```
FROM node:14.14.0-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

### Worker Dockerfile

```
FROM node:14.14.0-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

### Nginx Dockerfile

This is exactly the same as the nginx Dockerfile.dev file.
```
FROM nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf
```

