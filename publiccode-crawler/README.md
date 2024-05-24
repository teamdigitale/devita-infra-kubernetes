# Deploy publiccode-crawler Helm chart

To install (Helm >= v3.8.0 required for OCI image support):

```shell-session
# Use your GitHub token when asked for password
helm registry login ghcr.io -u api

kubectl apply -f secretproviderclass.yaml

# Get latest chart version from
# https://github.com/italia/publiccode-crawler/pkgs/container/publiccode-crawler%2Fcharts%2Fpubliccode-crawler

helm install
    publiccode-crawler \
    oci://ghcr.io/italia/publiccode-crawler/charts/publiccode-crawler \
    --version <LATEST_CHART_VERSION> \
    -f custom.yaml
```

to upgrade:

```shell-session
helm upgrade
    publiccode-crawler \
    oci://ghcr.io/italia/publiccode-crawler/charts/publiccode-crawler \
    --version <LATEST_CHART_VERSION> \
    -f custom.yaml
```

to remove:

```shell-session
helm uninstall publiccode-crawler
kubectl delete -f secretproviderclass.yaml
```
