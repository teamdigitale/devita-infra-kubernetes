# Developers Italia Crawler

To deploy the Developers Italia crawler do the following:

* Create a secret named *k8s-secrets-crawler* in the Azure keyvult, following this template:

```json
{
    "elastic-pass": "YOUR_ELASTICSEARCH_PASSWORD",
    "domains.yml": "[{host: gitlab.com}, {host: bitbucket.org}, {host: github.com, basic-auth: [YOUR_GITHUB_TOKEN]}]"
}
```

The GitHub security token can be created [here](https://github.com/settings/tokens).

Then, deploy the chart with helm (from the root directory of this repository):

```shell
helm install crawler crawler
```
