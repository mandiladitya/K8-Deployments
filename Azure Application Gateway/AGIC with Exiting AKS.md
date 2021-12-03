##### In you AKS Cluster Networking Tab > Enable Application Gatway
#####  It'll take approx 15 mins. to create
###### Create AGIC-Deployment.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: aspnetapp
  labels:
    app: aspnetapp
spec:
  containers:
  - image: "adityamandil317/flaskrestapi"
    name: aspnetapp-image
    ports:
    - containerPort: 5001
      protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  name: aspnetapp
spec:
  selector:
    app: aspnetapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5001

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aspnetapp
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          service:
            name: aspnetapp
            port:
              number: 80
        pathType: Exact
```
##### kubectl apply -f AGIC-Deployment.yaml
##### kubectl get all
##### kubectl get ingress
