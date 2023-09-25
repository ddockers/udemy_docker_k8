# Kubernetes

## Definitions
**Pod**
- One or more containers running together on one node 
- Basic unit of deployment
- Containers are always in pods

**Controller**
- For creating/updating pods and other objects
- Types include Deployment, ReplicaSet, StatefulSet

**Service**
- Network endpoint to connect to a pod

**Namespace**
- Filtered group of objects in a cluster



## Commands

Official cheat sheet: <https://kubernetes.io/docs/reference/kubectl/cheatsheet/>
Official kubectl for Docker Users cheat sheet: <https://kubernetes.io/docs/reference/kubectl/cheatsheet/>
Chearography: <https://cheatography.com/deleted-44122/cheat-sheets/kubectl/>
K8s Concepts: <https://cheatography.com/gauravpandey44/cheat-sheets/kubernetes-k8s/>

- Check if K8s is working:

```
kubectl version
```

- Create a single pod (on Nginx server)

```
kubectl run my-nginx --image nginx
```
The pod always has to be given a name. In this case, it's *my-nginx*.

- List pods:

```
kubectl get pods
```

- List all resources (in current namespace):

```
kubectl get all
```

![Imgur](https://i.imgur.com/ve42CtU.png)

- Unlike Docker, you can't create a container directly in K8s
- You create Pods (vaia CLI, YAML or API)
- K8s then creates the containers inside it
- **Kubelet** tells the *container runtime (Docker)* to create containers for you
- Every type of resource to run containers uses pods

- Create a deployment:

```
kubectl create deployment my-nginx --image nginx
```

Running `kubectl get pods` and `kubectl get all` displays the following:

![Imgur](https://i.imgur.com/B8Nvgst.png)

- Delete pod:

```
kubectl delete pod <name of pod>
```

- Delete deployment:

```
kubectl delete deployment <name of deployment>
```

- Add more replicas to a deployment:

```
kubectl scale deploy my-apache --replicas 2
```

This provides 2 replica sets total, not 2 in addition to what we already had.

- `kubectl scale` will change the deployment.my-apache record
- Controller Manager will see that only the replia count has changed
- It will change the number of pods in a ReplicaSet
- Scheduler sees a new pod is requested and assigns a node
- Kubelet sees a new pod and tells the container runtime to start httpd


## `kubectl get`

![Imgur](https://i.imgur.com/6vcmxpu.png)

![Imgur](https://i.imgur.com/n5LzeT8.png)

The `-o yaml` flag lists info about the deployment in YAML format.

The same can be done for each resource, e.g.:

```
kubectl get pods -o <name of pod>
```

The name of the pod can be found my running `kubectl get pods`.

`kubectl get` can only show one resource at a time.

## `kubectl describe`

`kubectl describe <name of deployment>` shows:
- Deployment summary
- ReplicaSet status
- Pod template
- Old/new ReplicaSet names
- Deployment events

![Alt text](image.png)

`kubectl describe` can also be used on pods and nodes.

## Watching

`kubectl get pods -w` lets you see realtime status/changes to resources.

Running the above command and then `kubectl gelete pod <name of pod>` in another terminal window shows this:

![Imgur](https://i.imgur.com/J8U9eNV.png)

The old pod terminates and a new pod starts up.

`kubectl get events --watch-only` shows only new events on the pod. 

Running the above command and deleting a pod in another terminal window shows this:

![Imgur](https://i.imgur.com/k1A3j88.png)

## Container Logs

- Get a container's logs (first container only)

```
kubectl logs deploy/<name of deploy>
```

- Follow new log entries, starting with the latest log line:

```
kubectl logs deploy/<name of deploy> --follow --tail 1
```

- Get a specific container's logs in a pod:

```
kubectl logs pod/<name of pod> -c <container>
```

The container can be found using the `describe` command.

- Get logs for all containers in a pod:

```
kubectl logs pod/<name of pod> --all-containers=true
```

- Get multiple pod logs:

```
kubectl logs -l app=my-apache
```