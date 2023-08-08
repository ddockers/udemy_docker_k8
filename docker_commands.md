# Manipulating Containers with the Docker Client

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

