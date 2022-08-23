# Install the external-secrets operator

Install the chart:

```console
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets \
    external-secrets/external-secrets \
    -n external-secrets \
    --create-namespace \
    --version 0.5.1 --disable-openapi-validation
```

(last line above is necessary to install on older Kubernetes versions).

Install the provider:

```console
kubectl -n external-secrets apply -f azure-kv-secret-store.yaml
```

## Usage

Assuming that your app store secrets in the Azure secret `k8s-secrets-my-app` in JSON format, e.g.:

```json
{
    "username": "fooser",
    "password": "barbar"
}
```

create a file `secret.yaml` using this template:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-app
spec:
  secretStoreRef:
    kind: ClusterSecretStore
    name: azure-kv-secret-store
  target:
    name: my-app-azure-kv
  dataFrom:
    - extract:
        key: k8s-secrets-my-app
```

and run `kubectl apply -f secret.yaml`. This will generate a k8s secret containing the keys in the original JSON secret, `username` and `password`.