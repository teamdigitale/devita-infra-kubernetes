# Developers Italia onboarding app

To deploy the onboarding app, please proceed as follows:

* Create the PVC. From the root folder of this repo:

```shell
kubectl apply -f storage/prod-pvc-onboarding.yaml
```

* Create a secret in the Azure keyvault named *k8s-secrets-onboarding*, formatted as following

```json
{
  "smtp-hostname": "YOUR_SMTP_HOSTNAME",
  "smtp-username": "YOUR_SMTP_USERNAME",
  "smtp-password": "YOUR_SMTP_PASSWORD"
}
```

* Install the helm-chart

```shell
helm install onboarding onboarding
```

## Codebase

The codebase is available [here](https://github.com/italia/developers-italia-onboarding).
