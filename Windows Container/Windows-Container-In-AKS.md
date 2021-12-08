#### Create a resource group
``` 
az group create -n win-aks -l westus2
```

#### Create the cluster, with the default linux nodepool
```
az aks create -g win-aks -n win-aks \
  --node-count 2 --ssh-key-value ~/.ssh/id_rsa.pub \
  --windows-admin-username nilfranadmin \
  --windows-admin-password superSecret123! \
  --network-plugin azure
  ```

#### Create a second nodepool using Windows
 ```
 az aks nodepool add -g win-aks \
  --cluster-name win-aks \
  --os-type Windows --name winnp \
  --node-count 2
  ```
  
  ```
  apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: win-webserver
  name: win-webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: win-webserver
  template:
    metadata:
      labels:
        app: win-webserver
      name: win-webserver
    spec:
     containers:
      - name: windowswebserver
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
        image: mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2019
     nodeSelector:
      kubernetes.io/os: windows
  ```
  
  -  The service definition isn’t any different for Windows vs Linux workloads.
 ```
 apiVersion: v1
kind: Service
metadata:
  name: win-webserver
  labels:
    app: win-webserver
spec:
  selector:
    app: win-webserver
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
 ```
