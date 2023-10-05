# Prometheus helm-chart

Prometheus provides an [official helm-chart](https://github.com/helm/charts/tree/master/stable/prometheus-operator), that is used for the installation on the Developers Italia infrastructure.

A [custom.yaml](custom.yaml) file is provided in this directory to customize the installation.

## Installation

These are the steps to install the chart:

* Create a new namespace named *monitoring*: `kubectl create namespace monitoring`

* Create a secret named *k8s-secrets-prometheus* in the Azure Keyvault from the source containted in `alertmanager-config.yaml` using:

```console
yq '.. style="flow"' alertmanager-config.yaml
```

Make sure you substitute both the *YOUR_SLACK_WEBHOOK* and *YOUR_SLACK_CHANNEL* variables with your own values.

* Synchronize the Azure Keyvault secret with Kubernetes

```shell
kubectl apply -f secrets.yaml
```

* Install Prometheus

```shell
helm install \
    -f custom.yaml \
    --namespace monitoring \
    --version 8.12.7 \
    prometheus stable/prometheus-operator
```

## Upgrade setup / configuration

* Optionally, modify your configuration, including: *custom.yaml*, *secrets.yaml*, or the secret your saved in the azure keyvault

* Optionally, If you modified the keyvault secret, either wait for some time to get the new secret synced locally, or delete the secret and re-create it.

* Upgrade the chart

```shell
helm upgrade \
    -f custom.yaml \
    --namespace monitoring \
    --version 8.12.7 \
    prometheus stable/prometheus-operator
```

## Access Prometheus

```shell
kubectl -n monitoring port-forward prometheus-prometheus-kube-prometheus-prometheus-0 9090
```

Then, access http://localhost:9090

## Access Alertmanager

```shell
kubectl port-forward --namespace monitoring svc/prometheus-kube-prometheus-alertmanager 9093
```

## Access Grafana

```shell
kubectl port-forward --namespace monitoring svc/prometheus-grafana 3000:80
```
