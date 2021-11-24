#### Ingress in AKS
```
- kubectl create ns ingress-poc
- helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
- helm repo update
- helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace --namespace ingress-poc
- kubectl get all -n ingress-poc
```
