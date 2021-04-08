# Developers Italia onboarding app

To deploy the onboarding app, please proceed as follows:

1. Create the PVC. From the root folder of this repo:

   ```shell
   kubectl apply -f storage/prod-pvc-onboarding.yaml
   ```

1. Create a secret in the Azure keyvault named *k8s-secrets-onboarding*,
   formatted as following:

   ```json
   {
   "smtp-hostname": "YOUR_SMTP_HOSTNAME",
   "smtp-username": "YOUR_SMTP_USERNAME",
   "smtp-password": "YOUR_SMTP_PASSWORD"
   }
   ```

1. Install the helm-chart

   ```shell
   helm install onboarding onboarding
   ```

1. (optional) Install the app in staging

   ```shell
   helm install onboarding-staging onboarding -f onboarding/values-staging.yaml
   ```

## Bring up test environment on Kubernetes

You can override default chart values and install the onboarding app using the development settings:

* Make sure the *k8s-secrets-onboarding* reflect the development SMTP server

* Deploy the chart:

```shell
helm install \
    --set onboarding.env.onboarding_environment="dev" \
    --set onboarding.env.onboarding_email_bcc="cc@test.it" \
    --set onboarding.env.onboarding_email_bcc="bcc@test.it" \
    --set onboarding.env.onboarding_email_override_recipient_addr="override@test.it" \
    onboarding \
    onboarding
```


## Codebase

The codebase is available [here](https://github.com/italia/developers-italia-onboarding).
