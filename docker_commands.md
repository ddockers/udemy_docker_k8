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