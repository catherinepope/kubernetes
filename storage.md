# Storing Data

As Pods are ephemeral, data is lost when they're replaced. Although you can use storage on a node, a Pod might be restarted on a different node and lose access to that storage. You need *clusterwide* storage, so Pods can access the same data from any node.

Kubernetes allows you to define different storage classes provided by your cluster. You can then request a specific storage class for your application.

Each container in a Pod has its own filesystem, which is constructed by Kubernetes.

The filesystem can be built from multiple sources. At a minimum, this includes the layers from the container images and the writable layer for the container.

If the app inside a container crashes and the container exits, the Pod starts a replacement. The new container starts with the filesystem from the container image and a *new* writable layer. Any data written by the previous container is gone.

- The filesystem of a Pod container has the lifecycle of the *container*, rather than the Pod. 
- When Kubernetes restarts a Pod, this actually means a replacement container.
- If your apps are writing data inside containers, this data isn't stored at the Pod level.

Kubernetes can build the container filesystem from other sources. For example, you can surface ConfigMaps and Secrets in filesystem directories.

You define a volume at Pod level that makes another storage source available. Then you mount this storage source into the container filesystem at a specified path.

Here's a simple volume spec:

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      volumeMounts:
        - name: data
            mountPath: /data
volumes:
  - name: data
    emptyDir: {}
```

Any data stored in an `EmptyDir` volume remains in the Pod between restarts, so replacement containers can access data written by their predecessors.

You can use `EmptyDir` volumes for any applications that use the filesystem for temporary storage. For example, the app might save API responses in a local file, because reading from disk is faster than repeating an API call.

`EmptyDir` volumes share the lifecycle of the Pod. If the Pod is replaced, the new Pod starts with an empty directory. 

If you want your data to persist between Pods, you need to mount other types of volume with their own lifecycles.

## Storing Data on a Node

The simplest storage option is to use a volume that maps to a directory on the node. When the container writes to the volume mount, the data is actually stored in a known directory on the node's disk.

`HostPaths` are specified as a volume in the Pod, which is mounted into the container filesystem in the usual way. 

The limitation with `HostPaths` is that data is physically stored on the node. If your Pod is restarted on a different node, it loses access to that data.

This example spec mounts a HostPath volume:

```
spec: 
  containers:
    - image: nginx:1.17-alpine
      name: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: cache-volume
          mountPath: /data/nginx/cache
  volumes:
    - name: cache-volume
      hostPath:
        path: /volumes/nginx/cache
        type: DirectoryOrCreate
```

This method extends the durability of the data beyond the lifecycle of the Pod to the lifecycle of the node's disk, provided replacement Pods always run on the same node.

One major problem with `HostPath` is that you can inadvertently give complete access to data on the node. You need to define subpaths.

For example:

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      volumeMounts:
        - name: node-root
          mountPath: /pod-logs
          subPath: var/log/pods
        - name: node-root
          mountPath: /container-logs
          subPath: var/log/containers
volumes:
  - name: node-root
    hostPath:
      path: /
      type: Directory
```

## Storing Clusterwide Data

Pods can access volumes from any node if the volume uses *distributed storage*.

Kubernetes supports many volume types, backed by distributed storage systems. For example, EKS clusters can use Elastic Block Store.

- Pods are an abstraction over the compute layer.
- Services are an abstraction over the network layer.
- *PersistentVolumes* (PV) and *PersistentVolumeClaims* are abstractions in the *storage layer*.


A **PersistentVolume** is a Kubernetes object that defines an available piece of storage. A cluster administrator can create a set of PersistentVolumes, each containing a volume spec for the underlying storage system.

For example:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01

spec:
  capacity:
    storage: 50Mi
  accessModes:
    - ReadWriteOnce # can be used by only one Pod
    nfs:
      servers: nfs.my.network
      path: "/kubernetes/volumes"
```

Pods can't access that PersistentVolume directly - they need to *claim* it, using a **PersistentVolumeClaim** (PVC). 

Kubernetes matches the PVC with a PV.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc # claim will be used by a specific app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi
storageClassName: "" # a blank class means a PV needs to exist.
```

If there is a match, the PVC is bound to the PV. Once a PV is claimed, it's not available for other PVCs.

If there's no matching PV for your PVC, the claim is still created, but it's not usable. The claim remains in the system, waiting for a PV to be created that meets its requirements.

A PVC needs to be *bound* before a Pod can use it.

The following example is a Pod spec with a PVC:

```
spec:
  containers:
    - name: db
      images: postgres:11.6-alpine
      volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: postgres-pvc
```

If you delete a Pod and the replacement uses the same PVC and the same PV, the original data will still be available.

### Dynamic Volume Provisioning

With dynamic provisioning, you create the PVC and the PV is created on demand by the cluster. Clusters can be configured with multiple storage classes that reflect the different volume capabilities available, in addition to a default storage class.

PVCs can specify the name of the storage class they want. If they want to use the default class, they just omit the storage class field in the claim spec.

You create storage classes as standard Kubernetes resources. In the spec, you define exactly how the storage class works with the following three fields:

- **provisioner** - the component that creates PVs on demand. Different platforms have different provisioners. For example, the provisioner in the default AKS storage class integrates with Azure Files to create new file shares.
- **reclaimPolicy** - defines what happens to the dynamically created volumes when the claim is deleted. The underlying volume can be deleted or retained.
- **volumeBindingMode** - determines whether the PV is created as soon as the PV is created, or not until a Pod that *uses* the PVC is created.

Here's an example of a PVC using a storage class:

```
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: kiamol
  resources:
    requests:
      storage: 100Mi
```