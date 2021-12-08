```
There is one other type ReplicationController but Kubernetes now favors 
Deployments as Deployments configure ReplicaSets to support replication.
```
## Deployments
It is a Kubernetes controller that matches the current state of your cluster to the desired state mentioned in the Deployment manifest. e.g. If you create a deployment with 1 replica, it will check that the desired state of ReplicaSet is 1 and current state is 0, so it will create a ReplicaSet, which will further create the pod. If you create a deployment with name counter, 
it will create a ReplicaSet with name `counter-<replica-set-id>`, which will further create a Pod with name `counter-<replica-set->-<pod-id>`

- Deployments are usually used for stateless applications.
- you can save the state of deployment by attaching a Persistent Volume to it and make it stateful, but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.
 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: counter
        image: "kahootali/counter:1.1"
        volumeMounts:
        - name: counter
          mountPath: /app/
      volumes:
      - name: counter
        persistentVolumeClaim:
          claimName: counter
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: counter
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: efs
```
<img src="https://miro.medium.com/max/875/1*m1BoqSmhxGeGHmeexJ_mZA.png" style="width:50%"></img>

- Rolling Update means that the previous ReplicaSet doesn’t scale to 0 unless the new ReplicaSet is up & running ensuring 100% uptime. 
- If an error occurs while updating, the new ReplicaSet will never be in Ready state, so old ReplicaSet will not terminate again ensuring 100% uptime in case of a failed update. In Deployments, you can also manually roll back to a previous ReplicaSet, if needed in case if your new feature is not working as expected.
----------------------------------------------------

### StatefulSets

- StatefulSet(stable-GA in k8s v1.9) is a Kubernetes resource used to manage stateful applications. It manages the deployment and scaling of a set of Pods, and provides guarantee about the ordering and uniqueness of these Pods.
- StatefulSet is also a Controller but unlike Deployments, it doesn’t create ReplicaSet rather itself creates the Pod with a unique naming convention. e.g. If you create a StatefulSet with name counter, it will create a pod with name counter-0, and for multiple replicas of a statefulset, their names will increment like counter-0, counter-1, counter-2, etc
```
Every replica of a stateful set will have its own state, and each of the pods will be creating its own PVC(Persistent Volume Claim). 
So a statefulset with 3 replicas will create 3 pods, each having its own Volume, so total 3 PVCs.
```
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: counter
spec:
  serviceName: "counter-app"
  selector:
    matchLabels:
      app: counter
  replicas: 1 
  template:
    metadata:
      labels:
        app: counter
    spec:      
      containers:
      - name: counter
        image: "kahootali/counter:1.1"  
        volumeMounts:
        - name: counter
          mountPath: /app/      
  volumeClaimTemplates:
  - metadata:
      name: counter
    spec:
      accessModes: [ "ReadWriteMany" ]
      storageClassName: efs
      resources:
        requests:
          storage: 50Mi
```
- Here, the logs are again starting from 1, as this pod has its own Volume, so it doesn’t read the file of 1st pod. And if we see the Persistent Volume Claims,their will be 3 claims created as we had scaled the replicas to 3.
<img src="https://miro.medium.com/max/875/1*t0TyiZMRyJCPOGg2XphzwA.png" style="width:50%"></img>

- StatefulSets don’t create ReplicaSet or anything of that sort, so you cant rollback a StatefulSet to a previous version. You can only delete or scale up/down the Statefulset
- StatefulSets are useful in case of Databases especially when we need Highly Available Databases in production as we create a cluster of Database replicas with one being the primary replica and others being the secondary replicas. The primary will be responsible for read/write operations and secondary for read only operations and they will be syncing data with the primary one.

- If the primary goes down, any of the secondary replica will become primary and the StatefulSet controller will create a new replica in account of the one that went down, which will now become a secondary replica.

<img src="https://miro.medium.com/max/875/1*dh5O8yRAbdXHjmVnu9IfzA.png" style="width:50%"></img>
In case if postgres-0 went down, now postgres-1 became the primary replica

-----------------------------------------------------------

### DaemonSet
- A DaemonSet is a controller that ensures that the pod runs on all the nodes of the cluster. If a node is added/removed from a cluster, DaemonSet automatically adds/deletes the pod.
```
Some typical use cases of a DaemonSet is to run cluster level applications like:
Monitoring Exporters: You would want to monitor all the nodes of your cluster so you will need to run a monitor on all the nodes of the cluster like NodeExporter.
Logs Collection Daemon: You would want to export logs from all nodes so you would need a DaemonSet of log collector like Fluentd to export logs from all your nodes.
```
- However, Daemonset automatically doesn’t run on nodes which have a taint e.g. Master. You will have to specify the tolerations for it on the pod.
Taints are a way of telling the nodes to repel the pods i.e. no pods will be schedule on this node unless the pod tolerates the node with the same toleration. The master node is already tainted by:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: counter-app
spec:  
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      name: counter-app
      labels:
        app: counter
    spec:
      tolerations: 
      - effect: NoSchedule
        operator: Exists
      containers:
      - name: counter
        image: "kahootali/counter:1.1"
        volumeMounts:
        - name: counter
          mountPath: /app/
      volumes:
      - name: counter
        persistentVolumeClaim:
          claimName: counter
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: counter
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: efs
```
- In terms of behavior, it will behave the same as Deployments i.e. all pods will share the same Persistent Volume.
<img src="https://miro.medium.com/max/875/1*xrAwDE0tFymV62BiAwDyGw.png" style="width:50%"></img>

- If you update a DaemonSet, it also performs RollingUpdate i.e. one pod will go down and the updated pod will come up, then the next replica pod will go down in same manner 
- If an error occurs while updating, so only one pod will be down, all other pods will still be up, running on previous stable version. Unlike Deployments, you cannot roll back your DaemonSet to a previous version.

------------------------
#### Reference : 
- https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4
