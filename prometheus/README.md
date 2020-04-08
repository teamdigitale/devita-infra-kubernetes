# Prometheus helm-chart

Prometheus provides an [official helm-chart](https://github.com/helm/charts/tree/master/stable/prometheus-operator), that is used for the installation on the Developers Italia infrastructure.

A [custom.yaml](custom.yaml) file is provided in this directory to customize the installation.

## Installation

These are the steps to install the chart:

* Create a new namespace named *monitoring*: `kubectl create namespace monitoring`

* Create a secret named *k8s-secrets-prometheus* in the Azure Keyvault. This is the format that should be used:

```yaml
{global: {resolve_timeout: 5m}, route: {group_by: [job], group_wait: 30s, group_interval: 5m, repeat_interval: 12h, receiver: 'null', routes: [{match: {alertname: Watchdog}, receiver: 'null'}, {match_re: {severity: ^(none|warning|critical)$}, receiver: SlackChannel}]}, receivers: [{name: 'null'}, {name: SlackChannel, slack_configs: [{api_url: 'YOUR_SLACK_WEBHOOK', channel: '#YOUR_SLACK_CHANNEL'}]}]}
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
kubectl -n monitoring port-forward prometheus-prometheus-prometheus-oper-prometheus-0 9090
```

Then, access http://localhost:9000

## Access Alertmanager

```shell
alertmanager_pod=$(kubectl get pods --namespace monitoring -l "app=alertmanager" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $alertmanager_pod 9093
```

## Access Grafana

```shell
graphana_pod=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $graphana_pod 3000
```
