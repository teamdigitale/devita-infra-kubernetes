# Deploy developers-italia-api Helm chart

To install (Helm >= v3.8.0 required for OCI image support):

```shell-session
# Use your GitHub token when asked for password
helm registry login ghcr.io -u api

kubectl create namespace developers-italia-api
kubectl -n developers-italia-api apply -f secrets.yaml

# Get latest chart version from
# https://github.com/italia/developers-italia-api/pkgs/container/developers-italia-api%2Fcharts%2Fdevelopers-italia-api

helm -n developers-italia-api install \
    developers-italia-api \
    oci://ghcr.io/italia/developers-italia-api/charts/developers-italia-api \
    --version <LATEST_CHART_VERSION> \
    -f custom.yaml
```

to upgrade:

```shell-session
helm -n developers-italia-api upgrade \
    developers-italia-api \
    oci://ghcr.io/italia/developers-italia-api/charts/developers-italia-api \
    --version <LATEST_CHART_VERSION> \
    -f custom.yaml
```

to remove:

```shell-session
helm -n developers-italia-api uninstall developers-italia-api
kubectl -n developers-italia-api delete -f secrets.yaml
kubectl delete namespace developers-italia-api
```

## Staging

Optionally you can install a staging deploy tracking the `main`
branch of developers-italia-api:

```shell-session
kubectl create namespace developers-italia-api-staging

kubectl -n developers-italia-api-staging apply -f secrets-staging.yaml

# Get latest chart version from
# https://github.com/italia/developers-italia-api/pkgs/container/developers-italia-api%2Fcharts%2Fdevelopers-italia-api

helm -n developers-italia-api-staging install \
    developers-italia-api-staging \
    oci://ghcr.io/italia/developers-italia-api/charts/developers-italia-api \
    --version <LATEST_CHART_VERSION> \
    -f custom-staging.yaml
```
