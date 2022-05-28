# Connecting Pods with Services

Most applications are distributed across multiple components, and you'd use a Pod for each component. In a typical microservice architecture, all those Pods need to communicated.

Kubernetes supports the standard networking protocols TCP and UDP. Both protocols use IP addressess to route traffic. However, Pods are ephemeral and IP addresses change when they are replaced. Kubernetes solves this problem with *Services*, a network address discovery mechanism.

Services support:

- Routing traffic between Pods.
- Routing external traffic into the Pods.
- Routing traffic from Pods to external systems.

A Kubernetes cluster has a built-in DNS server, which maps Service names to IP addresses. The Service is loosely coupled to the Pod using a label-selector approach.

This type of Service is an abstraction over a Pod and its network address, just as a Deployment is an abstraction over a Pod and its container.

The Service has its own IP address, which is static. When network requests are made to that address, Kubernetes routes it to the actual IP address of the Pod. The link between the Service and its Pods is achieved with a label selector, just like the link between Deployments and Pods.

Here is a simple specification for a Service:

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
```

In the example above, the selector links all Pods with an app label of `my-app`. The Service name - `my-service` - is used as the DNS domain name.

## Routing Traffic Between Pods

The default Service type in Kubernetes is *ClusterIP*. It creates a clusterwide IP address that Pods on any node can access.

The IP address works only within the cluster, so ClusterIP Services are useful only for communicating *between* Pods. This is useful where you have components that shouldn't be accessible outside of the cluster.

The following example specification creates a Service with the ClusterIP type:

```
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec:
  ports:
    - port: 80
  selector:
    app: numbers-api 
  type: ClusterIP

```

:note: As ClusterIP is the default Service type, it can be omitted from the specification. However, including the type makes it clearer.


### Key Points

- In Kubernetes, you deploy a Service resource and use the name of the Service as the domain name for components to communicate.