```
There is one other type ReplicationController but Kubernetes now favors 
Deployments as Deployments configure ReplicaSets to support replication.
```
## Deployments
It is a Kubernetes controller that matches the current state of your cluster to the desired state mentioned in the Deployment manifest. e.g. If you create a deployment with 1 replica, it will check that the desired state of ReplicaSet is 1 and current state is 0, so it will create a ReplicaSet, which will further create the pod. If you create a deployment with name counter, 
it will create a ReplicaSet with name `counter-<replica-set-id>`, which will further create a Pod with name `counter-<replica-set->-<pod-id>`

- Deployments are usually used for stateless applications.
- you can save the state of deployment by attaching a Persistent Volume to it and make it stateful, but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.
 
<script src="https://gist.github.com/kahootali/1de83f98b8cf0b5caf8c73fe1be0241c.js"></script>
