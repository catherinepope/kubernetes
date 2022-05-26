# Pods and Deployments

## Pods

Although Kubernetes orchestrates containers, *you* don't work with those containers directly. Every container belongs to a **Pod**, which is a Kubernetes object for managing one or more containers.

A Pod runs on a single node in the cluster. The Pod has its own virtual IP address, which is managed centrally by Kubernetes. This means Pods can communicate with each other across the network, even if they're running on different nodes.

Normally, you'd run only one container in each Pod. However, it's possible to run multiple containers, known as *sidecar* containers. In this case, all the containers in the Pod share the same network address.

It's important to note that Kubernetes *manages* Pods, it doesn't run the containers within them. You also need a *container runtime* installed on the node. Examples of container runtimes include:

- Docker
- containerd
- runC

The `container ID` in the Pod is a reference to your container runtime, in this case Docker.

![Output of kubectl describe pod, showing Docker ID](/images/docker-id.png)

### Creating Pods

Usually, you'd create Pods by providing an *application manifest* through a YAML file. However, you can also create a simple Pod through the command line.

The syntax is similar to running a container using Docker: you specify the container image you want to use, along with any other required parameters to determine the Pod's behaviour. For example:

```
kubectl run hello-kiamol --image=kiamol/ch02-hello-kiamol --restart=Never
```

The code above creates and runs a Pod called hello-kiamol with a single container using the `kiamol/ch02-hello-kiamol` Docker image from Docker Hub.

The `restart` flag tells Kubernetes to create just the Pod and no other resources.

Once the Pod is running, you can start interacting with it.

First, you can see all the Pods currently running:

```
kubectl get pods
```

The command above provides basic information about your Pod(s). For more information, use:

```
kubectl get pods -o wide
```

Once you know what Pods are running, you can get even more information about a specific Pod:

```
kubectl describe pod hello-kiamol
```

The `describe pod` command gives you lots of information, which will be useful later on.

Alternatively, you can use the `get pod` command and specify exactly what information you want to see. For example:

```
kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'
```

The code above returns the Docker ID of the first container in the Pod. If there were additional containers in the Pod, you could query them by changing the `containerStatuses` index. This example is using the JSONPath query language. You could use Go templates for more complex outputs.



#### Key Points

- Kubernetes doesn't run containers directly, this is done by the *container runtime*.
- Pods are an *abstraction* of containers. Kubernetes manages those Pods.
- You can interact with Pods using kubectl commands.
