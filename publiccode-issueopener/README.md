# Deploy publiccode-issueopener

```shell-session
kubectl apply -f secretproviderclass.yaml

helm install \
    publiccode-issueopener \
    . \
    -f values.yaml
```

to upgrade:

```shell-session
helm install \
    publiccode-issueopener \
    . \
    -f values.yaml
```

to remove:

```shell-session
helm uninstall publiccode-issueopener
kubectl delete -f secretproviderclass.yaml
```
