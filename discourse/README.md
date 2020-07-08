# Discourse helm-chart for forum.italia.it

Discourse provides an [official helm-chart](https://github.com/bitnami/charts/tree/master/bitnami/discourse), that is used for the installation on the Developers Italia infrastructure.

A [custom.yaml](custom.yaml) file is provided in this directory to customize the installation.

These are the steps to install the chart:

* Create a secret named *k8s-secrets-discourse* in the Azure Keyvault. This is the format that should be used:

```json
{
  "discourse-password": "YOUR-DISCOURSE-PASSWORD",
  "postgresql-postgres-password": "YOUR_POSTGRESQL_PASSWORD",
  "postgresql-password": "YOUR_POSTGRESQL_ROOT_PASSWORD",
  "postgresql-replication-password": "YOUR_POSTGRESQL_REPLICATION_PASSWORD",
  "redis-password": "YOUR_REDIS_PASSWORD"
}
```

* Synchronize the Azure Keyvault secret with Kubernetes

```shell
kubectl apply -f secrets.yaml
```

* Install Discourse

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update

helm upgrade \
    --install \
    --version 0.1.3 \
    -f custom.yaml \
    discourse bitnami/discourse
```
