# k8s

Install Prometheus & Traefik in eks

## Lets install Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090

## Lets install Traefik

kubectl create namespace traefik
helm install traefik traefik/traefik --namespace traefik
kubectl get pods -n traefik
helm install traefik traefik/traefik --namespace traefik --set dashboard.enabled=true
kubectl port-forward -n traefik svc/traefik 9000:9000

