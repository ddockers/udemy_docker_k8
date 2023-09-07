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


