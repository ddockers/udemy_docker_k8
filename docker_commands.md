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

To remove the containers permanently, the service needs to be removed.

- Remove a service:

```
docker service rm <service ID/name>
```

# Multipass

With Multipass, multiple Ubuntu VMs/nodes can be launched directly in GitBash.

- Start a Multipass instance:

```
multipass launch
```

- Launch a named instsnce:

```
multipass launch -n <name of instance>
```
- View Multipass instances:

```
multipass ls
```

![Imgur](https://i.imgur.com/Ju0o6jn.png) 

- Run commands inside an instance:

```
multipass exec <name of instance> <command>
```

![Imgur](https://i.imgur.com/QegZjP5.png)

- Open a shell inside an instance:

```
multipass shell <name of instance>
```

Docker needs to be installed on the instances. 

Full installation commands here: <https://get.docker.com/>

```
curl -fsSL https://get.docker.com -o install-docker.sh

sudo sh install-docker.sh
```

`sudo` needs to be used when running Docker commands. 

Initiate a swarm on one instance by running `sudo docker swarm init`.

![Imgur](https://i.imgur.com/MtMJ4Sy.png)

Otner nodes can be added to the swarm by running the command printed on the screen with `sudo`.


## Troubleshooting
**Error 1**
```
sudo docker swarm join-token worker

SWMTKN-1-1ofu1fryp147sgyic2mm5xhvvoodw1br5oyzzg4sqnyuib1yri-b4yqb9fwx7ocegl89ngkbc48a 10.0.2.15:2377
```
**Error 2**
```
Error response from daemon: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial tcp 10.0.2.15:2377: connect: connection refused"
```

```
docker swarm join --token SWMTKN-1-06p8u7imhx04bqvf9jvl2gvdmqaji4bxwqgpy4g6crvj6noof1-bekjs5l37frw9nr6f46457lo8 10.0.2.15:2377
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/swarm/join": dial unix /var/run/docker.sock: connect: permission denied
```
**Solution 1**
Create a YAML file with Docker installation instructions to use with `cloud-init`. `usermod -aG docker ubuntu` gives the user (ubuntu) permissions to run Docker.

**Script 1**
```
#cloud-config
package_update: true
package_upgrade: true
runcmd:
  - curl -sSL https://get.docker.com/ | sh
  - usermod -aG docker ubuntu
```

Run:
```
multipass launch -n sonic --cloud-init <path to>multipass_vm_scr.yml
```

**Error 3**
```
launch failed: The following errors occurred:
timed out waiting for initialization to complete
```

**Script 2**
```
#cloud-config
package_update: true
package_upgrade: true
runcmd:
  - curl -fsSL https://get.docker.com -o install-docker.sh
  - sudo sh install-docker.sh
  - usermod -aG docker ubuntu
```

**Error 4**
Same as above.

## Solution
In multipass_vm_scr.yml script, update/upgrade were removed and single quotes added.
```
runcmd:
  - 'curl -sSL https://get.docker.com/ | sh'
  - 'usermod -aG docker ubuntu'
```