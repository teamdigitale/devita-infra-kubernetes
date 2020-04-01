# Slackin for Developers Italia

To deploy Slackin, please proceed as follows:

* Create a secret in the Azure keyvault named *k8s-secrets-slackin*, formatted as following

```json
{
  "slack-coc": "",
  "slack-channels": "",
  "slack-subdomain": "developersitalia",
  "slack-api-token": "YOUR_SLACK_API_TOKEN",
  "google-captcha-secret": "YOUR_GOOGLE_CAPTCHA_SECRET",
  "google-captcha-sitekey": "YOUR_GOOGLE_CAPTCHA_SITEKEY"
}
```

* Install the helm-chart

```shell
helm install slackin slackin
```

## Codebase

The codebase is available [here](https://github.com/italia/developers-italia-slackin).
