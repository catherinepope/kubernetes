# Controllers

Pods are very clever, but they're not useful on their own. They are isolated instances of an application, and each Pod is allocated to *one* node. If that node goes offline, the Pod is lost and Kubernetes doesn't replace it. Even though you could run several Pods, Kubernetes might allocate them all to the same node. This is why you need *controllers* to orchestrate it for you.

A controller is a Kubernetes resource that manages other resources. It works with the Kubernetes API to:

- Watch the current state of the system.
- Compare the current state to the *desired* state.
- Make any necessary changes.

Kubernetes includes many controllers, but the main controller for managing Pods is the *Deployment*.

## Deployments

If a node goes offline and you lose a Pod, the Deployment creates a replacement Pod on another node. If you want to scale your Deployment, you specify the number of Pods and the Deployment controller runs them across multiple nodes.

You can create Deployment resources with kubectl, specifying the container image and any other configuration for the Pod. Kubernetes creates the Deployment, and the Deployment creates the Pod.

The following command creates a Deployment:

`kubectl create deployment hello-kiamol-2 --image=kiamol/ch02-hello-kiamol`

Kubernetes generates a name for the Pod, which is the name of the Deployment followed by a random suffix.

You created a Deployment, then that Deployment created the Pod. The Deployment specification described the Pod you wanted. 

As a controller, the Deployment:

- Checks with the Kubernetes API to see which resources are running.
- Realises the Pod doesn't exist.
- Use the Kubernetes API to create that Pod.

### Tracking Deployments with Labels

Kubernetes uses labels to keep track of Deployments. These labels are simple key-value pairs and you can add them to your own data.

For example, you might add a label to a Deployment with the name `release` and the value `20.04` to indicate this Deployment is from the 20.04 release cycle.

This command prints the labels added to the Pod by the Deployment:

`kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'`

Now you can filter the Pods with that matching label:

`kubectl get pods -l app=hello-kiamol-2`

Resources can have labels applied at creation and then added, removed, or edited during their lifetime.

Controllers use a label selector to identify the resources they manage. This means controllers don't need to maintain a list of all the resources they manage. Instead, they can find matching resources at any time by querying the Kubernetes API.

:warning: Be careful when editing labels for a resource - you could accidentally break the relationship between that resource and its controller.