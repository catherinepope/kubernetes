# Scaling with DaemonSets

In Kubernetes, the DaemonSet runs a single replica of a Pod on every node in the cluster. In can also run on a subset of nodes, if you add a selector in the spec.

A common use for DaemonSets is where you want to get information from every node and send it to a central collector.

DaemonSets are a good solution where you want high availability without the load requirements of many replicas on each node. For example, a reverse proxy. A single NGINX Pod can handle many thousands of concurrent connections. You don't need a lot of them, but you want to make sure there's one running on every node. This way, a local Pod can respond, wherever the traffic lands.

Here's an example spec for a DaemonSet:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pi-proxy
spec:
  selector:
    matchLabels:
      app: pi-proxy
template:
  metadata:
    labels:
      app: pi-proxy
spec: ...

```

The control loop watches for nodes joining the cluster. Any new nodes will be scheduled to start a replica Pod as soon as they join. 

DaemonSets have different rules from Deployments for creating replicas - their logic needs to watch node activity as well as Pod counts. However, they are still Pod controllers and will replace any Pods that are lost. 