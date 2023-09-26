# Creating ClusterIP Services

Two terminal windows are needed.

In one window, we'll watch the pods by running `kubectl get pods -w`.

**Window 2**
Start a simple http server using sample code.

```
kubectl create deploy httpenv --image=bretfisher/httpenv
```

**Window 1**

![Imgur](https://i.imgur.com/MTTPJt0.png)

**Window 2**
Scale it to 5 replicas.
```
kubectl scale deploy httpenv --replicas=5
```

**Window 1**
4 more pods have been created.

![Imgur](https://i.imgur.com/yAFkNXv.png)


## Creating the ClusterIP Service

Everything currently running can be seen by running `kubectl get all`:

![Imgur](https://i.imgur.com/MYE2Csl.png)

The `expose` command is used to create the service. ClusterIP is the default service created.

**Window 2**

```
kubectl expose deploy httpenv --port 8888
```

We're listing which deployment to expose and which port the deployment is listening on.

## Create a NodePort Service

This will be exposed on the host IP.

```
kubectl expose deploy httpenv --port 8888 --name httpenv-np --type NodePort
```

![Imgur](https://i.imgur.com/lJkWJEn.png)

Default ranges for NodePort are 30000-32767. 8888 is the port inside the container itself. 32115 is the port on the node exposed to the outside world.

When a NodePort is created, a ClusterIP endpoint is automatically created.

## Add a LoadBalancer Service

```
kubectl expose deploy httpenv --port 8888 --name httpenv-lb --type LoadBalancer
```

![Imgur](https://i.imgur.com/ZntMMxW.png)

![Imgur](https://i.imgur.com/zFQDsZJ.png)