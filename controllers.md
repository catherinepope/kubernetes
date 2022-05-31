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

Kubernetes uses *labels* to keep track of Deployments. These labels are simple key-value pairs and you can add them to your own data.

For example, you might add a label to a Deployment with the name `release` and the value `20.04` to indicate this Deployment is from the 20.04 release cycle.

This command prints the labels added to the Pod by the Deployment:

`kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'`

Now you can filter the Pods with that matching label:

`kubectl get pods -l app=hello-kiamol-2`

Resources can have labels applied at creation and then added, removed, or edited during their lifetime.

Controllers use a label selector to identify the resources they manage. This means controllers don't need to maintain a list of all the resources they manage. Instead, they can find matching resources at any time by querying the Kubernetes API.

:warning: Be careful when editing labels for a resource - you could accidentally break the relationship between that resource and its controller.

However, this can be a useful technique in debugging: removing a Pod from a controller so you can connect and investigate a problem. In the meantime, the controller starts a replacement, which means your app keeps running at the desired scale. 

Also, you could force a Deployment to adopt a Pod if the labels match. If there are now surplus Pods, the Deployment uses deletion rules to remove one.

### Viewing a Pod in a Deployment

As with individual Pods, you can use `port forward` to view an app that's running as part of a Deployment. This time, the Deployment selects one of its Pods as the target:

`kubectl port-forward deploy/hello-kiamol-2 8080:80`

In the example above, you're not interacting with the Pod directly, rather with the Deployment that's managing it.


### Deleting Pods

If you created a Pod with a Deployment, you can't delete it directly with kubectl. It's the Deployment's job to manage that resource, not you! 

The following command will delete all your Pods:

`kubectl delete pods --all`

But, any created by controllers, such as a Deployment, will spring back into life - that's exactly what Kubernetes is designed to do.

If you want to delete a resource that's managed by a controller, you must delete the controller instead. Removing a Deployment also deletes all its Pods.

Use the following command to view your Deployments:

`kubectl get deploy`

Then delete them all:

`kubectl delete deploy --all`

Use the following command to see what's still running in your cluster:

`kubectl get all`

It should just be the Kubernetes service.

The kubectl `delete` command can read a YAML file and delete the resources defined in thef ile. If you have multiple YAML files in a directory, you can use the directory name as the argument to `delete` (or `apply`) all the files therein:

```
kubectl delete -f todo-list/
kubectl apply -f my-app/
```

### Key Points

- Like containers, Pods are meant to be short-lived - that's why you need a Deployment to manage them.
- Deployments communicated with the Kubernetes API to maintain your desired state.
- Labels are used for tracking Deployments and filtering your Pods.