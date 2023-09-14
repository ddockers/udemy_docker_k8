# Manipulating Containers with the Docker Client
- Build and tag an image:
```
docker build -t <docker ID>/<repo/project name>:<version> .
```
e.g. `docker build -t ddoxton/tech241-node-app:latest .`

- Create and run a container using an image:
```
docker run <image name>
```
- Do the same, but override the default command:
```
docker run <image name> <command>
```
![Imgur](https://i.imgur.com/5iAit2r.png)

Some commands won't work with certain images. Busybox has `echo` and `ls` that exist in the file system image.

- Create, run and **name** an image (Nginx in this example):
```
docker container run --publish 80:80 --name webhost nginx
```
Running `docker container ls` shows that the container's name is now `webhost`.

![Imgur](https://i.imgur.com/Km5kemJ.png)

I can run commands specifically on this container more easily. For example:

```
docker container logs webhost

docker container top webhost

docker container port webhost

docker container rm -f webhost
```
- List all running containers:
```
docker ps
```
- Delete:
  - all stopped containers
  - all networks not being used by at least one container
  - all dangling images
  - all build cache
```
docker system prune
```
- Execute commands in a running container
```
docker exec -it <container id> <command>
```

`-it` allows us to provide input into the container.
![Imgur](https://i.imgur.com/usSk1HO.png)

- Opening a shell inside running container
```
docker exec -it <container id> sh
```
![Imgur](https://i.imgur.com/wHM1bBD.png)

- View the process list for one container:

```
docker container top <container name/ID>
```

- View JSON array of metadata for container startup:

```
docker container inspect <container name/ID>
```

- See stats for running containers:
  
```
docker container stats
```
![Imgur](https://i.imgur.com/e1AhIqV.png)

- Stop a running container:

```
docker container stop <container name/ID>
```

- Remove stopped container:

```
docker container rm <container name/ID>
```

# Networks
- View Docker networks:

```
docker network ls
```

![Imgur](https://i.imgur.com/0GeUwTT.png)

- View containers attached to each network:

```
docker network inspect <network name>
```

![Imgur](https://i.imgur.com/4eV6O6b.png)

In the above example, I have an Nginx container named `webhost` running on the `bridge` network.

- Create a new Docker network:

```
docker network create <name of new network>
```

![Imgur](https://i.imgur.com/AWDVSIm.png)

- Create and run a container on a specific Docker network:

```
docker container run -d --name new_nginx --network my_app_net nginx
```

Then `docker network inspect my_app_net` can be run to see the container running on that network.

![Imgur](https://i.imgur.com/ofhT2rW.png)

Another container with a new name can be run on the same network. For example, running `docker container run -d --name my_nginx --network my_app_net nginx` creates a container called `my_nginx` on the network `my_app_net`.

![Imgur](https://i.imgur.com/SYvVb8T.png)

# Docker Prune
- View space usage:

```
docker system df
```

- Clean up dangling images:

```
docker image prune
```

- Clean up everything not currently in use:

```
docker system prune
```

- Clean up entire system:

```
docker system prune -a
```

- Clean up volumes:

```
docker system prune --volumes
```

# Volumes

- View volumes:

```
docker volume ls
```

![Imgur](https://i.imgur.com/fx79GiD.png)

- Inspect a volume:

```
docker volume inspect <volume name>
```

![Imgur](https://i.imgur.com/BZfODMB.png)


- Name a container volume:

```
docker container run <...> -v <volume name>:<volume location>
```

The container volume location can be found under `Config` when running `docker container inspect <name of container>`, or it can be found on the Docker Hub documentation for the image.

![Imgur](https://i.imgur.com/3fZpifb.png)

![Imgur](https://i.imgur.com/oxu8xgG.png)

![Imgur](https://i.imgur.com/5Bk37e9.png)

# Docker Swarm
`Service` in a swarm replaces `docker run`.

- Initialise a Docker swarm:

```
docker swarm init
```

A service can now be started , and the swarm will orchestrate how it needs to be laid out.

For example:

```
docker service create alpine ping 8.8.8.8
```

![Imgur](https://i.imgur.com/9nshF6A.png)

![Imgur](https://i.imgur.com/6wJc0tk.png)

`REPLICAS` is how many are running/how many I have specified to run. The orchestrator makes those numbers match.

- Scale up replicas:

```
docker service update <service ID/name> --replicas <no. of replicas>
```

![Imgur](https://i.imgur.com/r5jLXH8.png)

- View replicas:

```
docker service ps <service ID/name>
```

![Imgur](https://i.imgur.com/iJ9BeZy.png)

The swarm will automatically launch a new service if a container that is currently running a service is removed.

![Imgur](https://i.imgur.com/IR8XqaR.png)