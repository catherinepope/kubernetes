# Object Ownership

Controllers use a label selector to find objects they manage. The objects themselves keep a record of their owner in a metadata field. When you delete a controller, its managed objects persist briefly.

Kubernetes runs a garbage collector process that looks for objects whose owner has been deleted, and deletes them too. 

This command shows which objects own the Pods:

```
kubectl get po -o custom-columns=NAME:'{.metadata.name}', OWNER:'{.metadata.ownerReferences[0].name}',OWNER_KIND:'{.metadata.ownerReferences[0].kind}'
```

And this command which objects own the ReplicaSets:

```
kubectl get rs -o custom-columns=NAME:'{.metadata.name}', OWNER:'{.metadata.ownerReferences[0].name}',OWNER_KIND:'{.metadata.ownerReferences[0].kind}'
```

Controllers track their dependents solely through the label selector. If you tinker with labels, you could break that relationship. 

If you're deleting a controller, you can stop cascading deletes. That removes the owner reference in the metadata for the dependents, so they don't get picked up by the garbage collector.

The following command deletes all controllers and Services with the `kiamol=ch06` label:

```
kubectl delete all -l kiamol=ch06
```

