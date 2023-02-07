# Deploy developers-italia-api Helm chart

To install (Helm >= v3.8.0 required for OCI image support):

```shell-session
# Use your GitHub token when asked for password
helm registry login ghcr.io -u api

kubectl apply -f secretproviderclass.yaml

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
helm upgrade \
    developers-italia-api \
    oci://ghcr.io/italia/developers-italia-api/charts/developers-italia-api \
    --version <LATEST_CHART_VERSION> \
    -f custom.yaml
```

to remove:

```shell-session
helm uninstall developers-italia-api
kubectl delete -f secretproviderclass.yaml
```

## Staging

Optionally you can install a staging deploy tracking the `main`
branch of developers-italia-api:

```shell-session
kubectl apply -f secretproviderclass-staging.yaml

# Get latest chart version from
# https://github.com/italia/developers-italia-api/pkgs/container/developers-italia-api%2Fcharts%2Fdevelopers-italia-api

helm install \
    developers-italia-api-staging \
    oci://ghcr.io/italia/developers-italia-api/charts/developers-italia-api \
    --version <LATEST_CHART_VERSION> \
    -f custom-staging.yaml
```
