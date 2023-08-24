# Install origin-ca-issuer

```console
VERSION="v0.6.1"
kubectl apply -f https://raw.githubusercontent.com/cloudflare/origin-ca-issuer/${VERSION}/deploy/crds/cert-manager.k8s.cloudflare.com_originissuers.yaml
kubectl create ns origin-ca-issuer
# Clone https://github.com/cloudflare/origin-ca-issuer somewhere
helm -n origin-ca-issuer install origin-ca-issuer <clone_dir>/deploy/charts/origin-ca-issuer \
    -f custom.yaml
```

```console
kubectl create secret generic \
    cloudflare-origin-ca-key \
    --from-literal key=v1.0-FFFFFFF-FFFFFFFF
```
