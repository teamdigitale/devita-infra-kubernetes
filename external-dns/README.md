# Install external-dns for Cloudflare

```console
kubectl create ns external-dns
kubectl -n external-dns apply -f secretproviderclass.yaml
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns
helm -n external-dns install external-dns-cloudflare \
    external-dns/external-dns \
    -f custom-cloudflare.yaml
```
