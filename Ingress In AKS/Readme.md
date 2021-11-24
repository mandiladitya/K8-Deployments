#### Ingress in AKS
```
- kubectl create ns ingress-poc
- helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
- helm repo update
- helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace --namespace ingress-poc
- kubectl get all -n ingress-poc
- kubectl apply -f aks-one-deploy.yaml -n ingress-poc
- kubectl apply -f aks-two-deploy.yaml -n ingress-poc
- kubectl apply -f ingress-file.yaml -n ingress-poc
- kubectl get all -n ingress-poc
```
