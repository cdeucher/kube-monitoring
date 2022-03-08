
# This project aims to provide a simple EFK example for testing.

## Install Elasticsearch Cluster
```bash
$ cd elasticsearch
$ helm repo add elastic https://helm.elastic.co
$ helm install elasticsearch-multi-master elastic/elasticsearch -f ./master.yaml --version 7.15.0
$ helm install elasticsearch-multi-data elastic/elasticsearch -f ./data.yaml --version 7.15.0
$ helm install elasticsearch-multi-client elastic/elasticsearch -f ./client.yaml --version 7.15.0
```
## Install Kibana
```bash
$ cd kibana
$ kubectl apply -f .
```

## Install Fluentd
```bash
$ cd fluentd
$ kubectl apply -f .
```

## Expose Kibana service
```bash
$ kube port-forward service/kibana 5601
```
# This code is based on the following article:
+ https://phoenixnap.com/kb/elasticsearch-kubernetes
+ https://github.com/elastic/helm-charts/blob/main/elasticsearch/README.md
