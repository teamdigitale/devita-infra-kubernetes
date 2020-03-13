# Kibana helm-chart

Kibana provides an [official helm-chart](https://github.com/elastic/helm-charts/tree/master/kibana), that is used for the installation on the Developers Italia infrastructure.

A [custom.yaml](custom.yaml) file is provided in this directory to customize the installation.

These are the steps to install the chart:

* Create a secret named *k8s-secrets-kibana* in the Azure Keyvault. This is the format that should be used:

```json
{
  "elasticsearch-username": "YOUR_ES_USERNAME",
  "elasticsearch-password": "YOUR_ES_PASSWORD",
}
```

* Synchronize the Azure Keyvault secret with Kubernetes

```shell
kubectl apply -f secrets.yaml
```

* Install Kibana

```shell
helm repo add elastic https://helm.elastic.co

helm repo update

helm install -f custom.yaml kibana elastic/kibana
```
