# Multicontainer Pods

The Pod is a virtual environment that creates a shared networking and filesystem space for one or more containers. When one container takes an action, the other containers see it and can react to it.

You can't have Linux and Windows containers running in the same Pod (yet), because Linux containers need to run on a Linux node and Windows containers on a Windows node.

Containers in a Pod share the network, so each container shares the IP address of the Pod. Multiple containers can receive external traffic, but they need to listen on different ports. Containers within the Pod can communicate using the localhost address.

Here's an example of a multicontainer Pod spec:

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      volumeMounts:
        - name: data
          mountPath: /data-rw
    - name: file-reader
      image: kiamol/ch03-sleep

      volumeMounts:
        - name: data
          mountPath: /data-ro
          readOnly: true
  volumes:
    - name: data
        emptyDir: {}

```

In the example above, both containers can access the volume mount, but only data-rw can write to it. An empty directory provides a simple scratch pad that all containers can access.

This command shows the container names:

```
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'
```

You can't print logs at a Pod level - you need to specify a container.

A Pod is a single unit and it should be used for a single component of your app. Additional containers (*sidecars*) can be used to *support* the app container, but you shouldn't be running different apps in the same Pod. Otherwise, you can't update, scale, and manage those components independently.

## Init Containers

*Init containers* run *before* the app container to set up part of the environment.

- You can have multiple init containers defined for each Pod.
- They run in sequence, in the order in which they appear in the Pod spec.- Each init container needs to complete successfully before the next one starts.
- All init containers must complete successfully before the Pod containers are started.
- App containers start in parallel once all init containers have completed.
- All app containers need to be ready before the Pod is considered to be ready.

A major use case is for an init container to write data that prepares the environment for the application container.

Here's an example:

```
spec:
  initContainers:
    - name: init-html
      image: kiamol/ch03-sleep
      command: ['sh', '-c', "echo '<!DOCTYPE html...' > /data/index.html"]
      volumeMounts:
    - name: data
      mountPath: /data
```

You might use an init container to set up the application environment using tools you don't want to be present in the running application. For example, an init container can use a Docker image with the git command line installed and clone a repository into the shared filesystem. The app container can then access the files without you having to set up the git client in your app image.

This command checks the init containers:

```
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.initContainerStatuses[*].name}'
```

Even if there are multiple containers, Pods are still a single unit of compute. Pods aren't ready until *all* the containers are ready, and Services only send traffic to Pods that are ready. Adding sidecars and init containers adds to your application's failure modes.

- If a Pod with init containers is replaced, the new Pod runs all the init containers again. You must ensure your init logic can be run repeatedly.
- If you deploy a change to the init container image(s) for a Pod, that restarts the Pod. Init containers all execute again, and app containers are replaced.
- If you deploy a Pod spec change to the app container image(s), the app containers are replaced, but the init containers are not executed again.
- If an application container exits, the Pod recreates it. Until the container is replaced, the Pod is not fully running and won't receive Service traffic.

Although Pod containers have a shared network and can share parts of the filesystem, they can't access each other's processes - the container boundary provides compute isolation. In some cases, you want your sidecar to access the processes in the application container, either for interprocess communication, or so the sidecar can fetch metrics about the app process.

You can enable this access with the `shareProcessNamespace: true` setting in the Pod spec. Now every container in the Pod shares the same compute space and can see each other's processes.






