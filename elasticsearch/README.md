# Elastic search helm-chart

Elasticsearch provides an [official helm-chart](https://github.com/elastic/helm-charts/tree/master/elasticsearch), that is used for the installation on the Developers Italia infrastructure.

A [custom.yaml](custom.yaml) file is provided in this directory to customize the installation.

More on Elasticsearch security settings can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html).

These are the steps to install the chart:

* Create a secret named *k8s-secrets-elasticsearch* in the Azure Keyvault. This is the format that should be used:

```json
{
  "username": "YOUR_ES_USERNAME",
  "password": "YOUR_ES_PASSWORD",
}
```

* Generate the *pfx* certificate for Elasticsearch and load it under the certificates section on the Azure keyvault, using name *k8s-secrets-elasticsearch-certs*

```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout elasticsearch.key -out elasticsearch.crt

openssl pkcs12 -export -out elasticsearch.pfx -inkey elasticsearch.key -in elasticsearch.crt
```

* Synchronize the Azure Keyvault secret with Kubernetes

```shell
kubectl apply -f secrets.yaml
```

* Install Elasticsearch

```shell
helm repo add elastic https://helm.elastic.co

helm repo update

helm install -f custom.yaml --version 7.6.1 elasticsearch elastic/elasticsearch
```
