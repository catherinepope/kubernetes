# Overview

Kubernetes runs and manages (or *orchestrates*) large-scale applications in containers. It's responsible for:

- Starting your containerized applications.
- Rolling out updates.
- Maintaining service levels.
- Scaling to meet demand.
- Securing access.

The main features of Kubernetes are:

- **Container Orchestration** - manage containers at scale across multiple servers.
- **Application Reliability** - create reliable, self-healing, and scalable applications.
- **Automation** - automate the management of your container apps to increase efficiency and reduce errors.

The two core concepts in Kubernetes are:

- The **API**, which you use to *define* your applications.
- The cluster, which *runs* your applications.

## The Cluster

A cluster is a set of individual servers that have all been configured to use a container runtime such as Docker. Kubernetes then joins these servers into a single logical unit, the *cluster*.

## Nodes

The servers within the cluster are called **nodes**. Some nodes run the Kubernetes API, while other run application workloads. Regardless of their role, all nodes run containers.

The Kubernetes API runs in containers on Linux nodes, but the cluster can include other platforms, such as Windows apps in containers.

As a cluster administrator, you can add nodes, take nodes offline for servicing, or roll out a Kubernetes upgrade across the cluster.

However, if you're using a managed service such as Elastic Kubernetes Service (EKS), you'll usually interact with the cluster as a single entity - you won't see the underlying nodes.

## Running Applications

You define your apps in YAML files (*application manifests*) and send those files to the Kubernetes API, using a command-line tool. Kubernetes reviews your request in the YAML and compares it with what's already running in the cluster. Kubernetes then makes any changes necessary to achieve your *desired state*. This could include:

- Updating a configuration.
- Removing containers.
- Creating new containers.

To ensure high availability, containers are distributed across the cluster. Your containers can then communicate with each other using virtual networks managed by Kubernetes, even if they're on different nodes.

It's your responsibility to define the structure of an application, but Kubernetes ensures it keeps running. If a node in your cluster goes offline and takes containers with it, Kubernetes leaps into action and starts replacement containers on other nodes. If an application container becomes unhealthy, Kubernetes can restart it. If a component is struggling with a heavy load, Kubernetes can start extra instances of that component in new containers.

In short, Kubernetes helps you build a scalable, portable, self-healing applications.

## Storage

The cluster includes a distributed database, where you can securely store:

- Configuration files for your applications.
- Secrets, such as API keys and connection credentials.

Kubernetes delivers this stored data to your containers so you can apply the correct configuration.

Kubernetes also provides storage that's physically stored on disks in the cluster node or on a shared storage systems. This means you can maintain *stateful* data outside of containers so it persists even if those containers are deleted or restarted. 

## Kubernetes Resources and Objects

Although Kubernetes runs containers, it wraps them in other objects (known as *resources*) to support scaling, rolling upgrades, and more complex deployment patterns. These resources include:

- Pods
- ReplicaSets
- Deployments

*Services* are Kubernetes objects for managing network access. They can send external traffic to containers or between containers within the cluster.

*Volumes* manage storage.

*ConfigMaps* store non-confidential data in key-value pairs.

*Secrets* store small amounts of sensitive data, such as passwords, tokens, or keys.