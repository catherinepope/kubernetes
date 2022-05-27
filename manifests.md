# Application Manifests

- Manifests are a complete description of your app.
- This means you can use them to recreate the same deployment on any Kubernetes cluster.
- They can be versioned and tracked in source control.
- Manifests can be written in JSON or YAML. 
  
Although JSON is the native language of the Kubernetes API, YAML is preferred because:

- It's easier to read.
- You can define multiple resources in the same file.
- You can add comments in the specification.

## Pod Manifests

This is a simple manifest, defining a single Pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-kiamol-3
spec:
  containers:
    - name: web
      image: kiamol/ch02-hello-kiamol
```

This is more detailed than a kubectl `run` command. However, the big advantage of the application manifest is that it's *declarative*. kubectl `run` and `create` are *imperative* operations - you're telling Kubernetes to do something. Manifests are *declarative* - you tell Kubernetes what you want, then Kubernets makes that happen.

To deploy apps from manifest files, you use the `apply` command. This tells Kubernetes to apply the configuration in that file to the cluster:

`kubectl apply -f pod.yaml`

This manifest specifies there should be a Pod called `hello-kiamol-3`. As a Pod with that name didn't already exist, Kubernetes created it.

kubectl can read the contents of any manifest file with a public URL, for example, from GitHub:

`kubectl apply -f https://raw.githubusercontent.com/sixeyed/kiamol/master/ch02/pod.yaml`

## Deployment Manifests

When you define a Deployment in a manifest, one of the required fields is the specification of the Pod that your Deployment should run.

That Pod specification is the same API for defining a Pod on its own, so the Deployment definition is effectively a composite.

This is a simple Deployment manifest:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kiamol-4
spec:
  selector:
    matchLabels:
      app: hello-kiamol-4
  template:
    metadata:
      labels:
        app: hello-kiamol-4
    spec:
      containers:
        - name: web
          image: kiamol/ch02-hello-kiamol
```

As before, apply the Deployment manifest:

`kubectl apply -f deployment.yaml`

To fine Pods managed by the new Deployment, filter by label:

`kubectl get pods -l app=hello-kiamol-4`

As the app grows in complexity, you can use the manifest to specify factors such as the number of replicas, CPU and memory limits, and where it writes data.
