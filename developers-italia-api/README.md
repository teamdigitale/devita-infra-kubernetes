# Deploy developers-italia-api Helm chart

To install:

```shell-session
kubectl create namespace developers-italia-api
kubectl -n developers-italia-api apply -f secrets.yaml
helm -n developers-italia-api install \
    developers-italia-api \
    oci://ghcr.io/italia/charts/developers-italia-api \
    -f custom.yaml
```

to upgrade:

```shell-session
helm -n developers-italia-api upgrade \
    developers-italia-api \
    oci://ghcr.io/italia/charts/developers-italia-api \
    -f custom.yaml
```

to remove:

```shell-session
helm -n developers-italia-api uninstall developers-italia-api
kubectl -n developers-italia-api delete -f secrets.yaml
kubectl delete namespace developers-italia-api 
```
