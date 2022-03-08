

### Install Helm repositories
```bash
# add grafana/prometheus Helm repo
 helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
 helm repo add grafana https://grafana.github.io/helm-charts
 helm repo add bitnami https://charts.bitnami.com/bitnami
 helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
 helm repo update
```

### Install Prometheus and share access
```bash
 kubectl create namespace prometheus
 helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"


# Minikube
 kubectl create namespace prometheus
 helm install prometheus prometheus-community/prometheus \
    --namespace prometheus
 kubectl get all -n prometheus
 kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
# http://localhost:8080/targets
```

### Install Grafana
```bash
 kubectl create namespace grafana
 helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='gr!sAWSome' \
    --values grafana.yaml \
    --set service.type=LoadBalancer
 export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
 echo "http://$ELB"


# Minikube
 kubectl create namespace grafana
 helm upgrade grafana grafana/grafana \
    --namespace grafana --values values.grafana.yaml \
    --set service.type=ClusterIP
 kubectl port-forward -n grafana service/grafana 9090:80
# http://localhost:9090
```

### Setup Nginx-ingress
```bash
 kubectl create namespace nginx-ingress
 helm install nginx-ingress --namespace grafana \
    --namespace nginx-ingress --values values.nginx-ingress.yaml \
    ingress-nginx/ingress-nginx
```

### Edit cm prometheus
```bash
kube get cm prometheus-server -n prometheus -o yaml > prometheus.yaml
# kube edit cm prometheus-server -n prometheus

# Add ingress-nginx job to the prometheus yaml file:
```
```yaml
    - job_name: ingress-nginx-pods
      scrape_interval: 15s
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_port_number]
        action: keep
        regex: 10254
      kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
          - nginx-ingress
        selectors:
        - role: "pod"
          label: "prom=scrape"
      
```
```bash
kube apply -f prometheus.yaml
kube rollout restart deployment.apps prometheus-server -n prometheus
# kube delete pod/$(kube get pods -n prometheus | grep -Eo "prometheus-server[a-z0-9\-]+") -n prometheus  

kubectl port-forward service/nginx-ingress-ingress-nginx-controller 9090:80 -n nginx-ingress
```

### Setup Grafana Dashboards
```bash
# admin password
 kubectl get secret grafana -n grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# Import Dashboards
# - Grafana -> Import
#   - 3119
#   - 6417

 kubectl create configmap cluster-monitoring -n grafana --from-file="dashboards/cluster_monitoring.json"
 kubectl patch configmap cluster-monitoring -n grafana -p '{"metadata":{"labels":{"grafana_dashboard":"1"}}}'

 kubectl create configmap app-monitoring -n grafana --from-file="dashboards/app_monitoring.json"
 kubectl patch configmap app-monitoring -n grafana -p '{"metadata":{"labels":{"grafana_dashboard":"1"}}}'
 
 kubectl create configmap grafana-nginx-ingress -n grafana --from-file="dashboards/grafana-nginx-ingress.json"
 kubectl patch configmap grafana-nginx-ingress -n grafana -p '{"metadata":{"labels":{"grafana_dashboard":"1"}}}' 
```

### Clean-Up
```bash
helm uninstall prometheus --namespace prometheus
kubectl delete ns prometheus

helm uninstall grafana --namespace grafana
kubectl delete ns grafana

helm uninstall nginx-ingress --namespace nginx-ingress
kubectl delete ns nginx-ingress
```


### Links
- https://klaushofrichter.medium.com/ingress-nginx-metrics-on-grafana-k3d-84dc48374869
- https://faun.pub/grafana-dashboards-as-kubernetes-configmaps-959fd3159266

