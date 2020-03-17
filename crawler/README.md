# Developers Italia Crawler

To deploy the Developers Italia crawler do the following:

* Create a secret named *k8s-secrets-crawler* in the Azure keyvult, following this template:

```json
{
    "elastic-pass": "YOUR_ELASTICSEARCH_PASSWORD",
    "domains.yml": "[{host: gitlab.com}, {host: bitbucket.org}, {host: github.com, basic-auth: [YOUR_GITHUB_TOKEN]}]"
}
```

> NOTE: the basic-auth parameter for each code hosting platform is optional, although strongly suggested to avoid rate-limiting issues.

The GitHub security token can be created [here](https://github.com/settings/tokens).

Then, deploy the chart with helm (from the root directory of this repository):

```shell
helm install crawler crawler
```

## Codebase

The codebase is available [here](https://github.com/italia/developers-italia-backend).
