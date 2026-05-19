# Monitoring stack — kube-prometheus-stack

Stack di osservabilità del cluster `developers-italia-prod`: Prometheus +
Alertmanager + Grafana + Prometheus Operator + kube-state-metrics +
node-exporter, installato tramite il chart Helm
[`prometheus-community/kube-prometheus-stack`](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

- **Release name**: `kube-prometheus-stack`
- **Namespace**: `monitoring`
- **Chart pinned**: `prometheus-community/kube-prometheus-stack 85.2.0`
  → prometheus-operator `v0.90.1`, Grafana `v13.0.0`, kube-state-metrics,
  prometheus-node-exporter.
- **Values**: [`custom.yaml`](custom.yaml) — fonte di verità; ogni modifica
  passa di qui (`git` + `helm upgrade`).
- **Ingress Grafana**: oggetto `Ingress` autonomo `monitoring/grafana`
  (`https://grafana.innovazione.gov.it`), **gestito fuori da questo
  chart** — l'upgrade non lo tocca.
- **Sidecar dashboard/datasource**: `kiwigrid/k8s-sidecar` (sottochart
  Grafana). Pesca i ConfigMap con label `grafana_dashboard=1` /
  `grafana_datasource=1` (es. dal chart `postgres-backup` in `backups`).
- **Alertmanager config**: via CRD `AlertmanagerConfig` distribuite per
  namespace (matcherStrategy `None`); nessun secret CSI/KV.

## Aggiungere il repo Helm (una tantum)

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Installazione iniziale

```sh
kubectl create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --version 85.2.0 -f custom.yaml
```

## Aggiornare i values (no chart bump)

```sh
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --version 85.2.0 -f custom.yaml
```

## Upgrade del chart (procedura major)

Il chart **non aggiorna i CRD su `helm upgrade`**: vanno applicati a mano
in modalità *server-side* per la versione di operator target prima del
`helm upgrade`. La lista canonica delle CRD per ogni versione operator è
nella [UPGRADE.md](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/UPGRADE.md)
del chart.

### CRD apply (target operator v0.90.1)

```sh
OP=v0.90.1
BASE=https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/$OP/example/prometheus-operator-crd
for crd in alertmanagerconfigs alertmanagers podmonitors probes \
           prometheusagents prometheuses prometheusrules \
           scrapeconfigs servicemonitors thanosrulers; do
  kubectl apply --server-side --force-conflicts \
    -f $BASE/monitoring.coreos.com_${crd}.yaml
done
```

### Helm diff + upgrade

```sh
helm diff upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --version 85.2.0 -f custom.yaml
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --version 85.2.0 -f custom.yaml
```

### Validazione post-upgrade

```sh
kubectl -n monitoring rollout status deploy/kube-prometheus-stack-operator
kubectl -n monitoring rollout status deploy/kube-prometheus-stack-grafana
kubectl -n monitoring rollout status sts/prometheus-kube-prometheus-stack-prometheus
kubectl -n monitoring rollout status sts/alertmanager-kube-prometheus-stack-alertmanager
kubectl -n monitoring get prometheusrules,servicemonitors,alertmanagerconfigs -A | wc -l
```

Quindi su Grafana: login admin, datasources Prometheus e Loki UP,
dashboard `postgres-backup` carica; su Prometheus `/targets` i target up;
su Alertmanager le rule caricate.

### Rollback

`helm rollback kube-prometheus-stack <revision>` ripristina i workload
del chart **ma non i CRD** (per un rollback completo riapplicare a mano
le CRD della release precedente). I PVC (Prometheus 50 GiB, Alertmanager
5 GiB, Grafana 10 GiB) sono preservati attraverso upgrade/rollback.

## Accessi locali (port-forward)

```sh
# Prometheus
kubectl -n monitoring port-forward svc/kube-prometheus-stack-prometheus 9090:9090
# Alertmanager
kubectl -n monitoring port-forward svc/kube-prometheus-stack-alertmanager 9093:9093
# Grafana (ingress pubblico: https://grafana.innovazione.gov.it)
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```
