# Kubernetes configuration scripts for Developers Italia

The repository contains the Kubernetes configurations used to setup the Developers Italia Kubernetes resources after they've been provisioned with [Terraform](https://github.com/teamdigitale/devita-infra-tf-live).

## About Developers Italia

More informations about Developers Italia can be found on the [Digital Transformation Department website](https://innovazione.gov.it/it/progetti/developers-italia/)

## Tools references

The repository makes mainly use of the following tools:

* [Kubernetes](https://kubernetes.io/)
* [Helm 3](https://helm.sh/)

## How to use this repository and its tools

The repository is a collection of scripts to configure Kubernetes resources (i.e. containers, services, nginx ingress rules...).
The Kubernetes cluster and any other Azure cloud resource should be previously provisioned using [these](https://github.com/teamdigitale/devita-infra-tf-live) Terraform scripts.

To setup the Kubernetes resources you should have full access to it with administrative privileges.

## Folder structure

Each folder generally represents an application, except a few of them that instead realize specific functions:

* **system** - contains system services (look at the *deploy system services* paragraph below)

* **storage** - contains the PersistentVolumeClaim configurations (PVCs) used by some of the applications to work. Inside, each PVC configuration is named as the chart that makes use of it

* **configs** - helm configuration value files that extend the basic helm charts default values. Each file generally represents a deployment environment (i.e. dev, prod, ...)

## Prerequisites

1. Ask your System Administrator for a personal Azure account to access the existing deployment
2. Install the [Azure CLI tool](https://github.com/Azure/azure-cli). Once installed, try to login with `az login`. A web page will be displayed. Once the process completes you should be able to use the az tool and continue
3. Install and setup [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
4. Install and setup the [helm 3](https://helm.sh/docs/using_helm/#installing-helm)
5. Make sure you're on the correct azure subscription (`az account set -s YOUR-SUBSCRIPTION`)

>NOTE: You can either ask your subscription id to your administrator or view it online from [portal.azure.com](portal.azure.com).

6. Download the K8S cluster configuration (`az aks get-credentials -n AKS_CLUSTER_NAME -g RESOURCE_GROUP`)

>NOTE: You can either ask your resource group and AKS cluster name to your administrator or view it online from [portal.azure.com](portal.azure.com).

7. You may have more than one cluster configuration on your computer.

* To see your contexts: `kubectl config get-contexts`
* To use your context: `kubectl config user-context CONTEXT_NAME`

  More info [here](https://kubernetes-v1-4.github.io/docs/user-guide/kubectl/kubectl_config_use-context/)

8. At this point you should be able to see the available PODs of the cluster, if any: `kubectl get pods`, and the helm deployments (`helm ls`).

## Deploy Kubernetes Resources

The paragraph explains how deploy and manage Kubernetes resources.

### Deploy system services

Following, are the instructions to deploy the system services needed by all other applications to work.

> **WARNING:** The following commands should be generally run once, while setting up the cluster the first time. Make sure this is your case before proceeding. If you run them anyway, nothing bad should happen, since all of them should be idempotent.

#### Groups and role bindings

Apply the groups and role bindings to bind the Azure groups to the Kubernetes groups, thus allowing specific users to perform specific actions. Before applying the configuration check that the group object ids reflect the ones configured in Azure. Then:

```shell
kubectl apply -f system/prod-azure-aad-cluster-roles.yaml
```

#### Storage and data persistence

By default Azure uses two *storage classes* to provide data persistence functionalities: *default* and *managed-premium*. These are automatically configured by Azure at setup time. By default, both classes do not support dynamic storage resizes, and their reclaim policy is set to *Delete*, which would cause data to be deleted when a persistent volume claim gets deleted.

##### Deploy the Azure Disk custom Storage Class

The *Azure disk custom storage class* implements all the features provided by both default storage classes, while also enabling dynamic storage resizes and setting the reclaim policy to *Retain*, thus preserving managed disks, even when a PVC gets deleted.

To deploy the custom storage class, run:

```shell
kubectl apply -f system/common-azure-disk-sc-custom.yaml
```

>Note: The limitation of disk type storage classes is that disks can be attached only to one pod at the time.

##### Deploy Azure Files Storage Class and related role-based access control (RBAC)

The Azure Files storage class is slower than the Azure disk storage classes mentioned above but it can be sometimes useful when multiple containers need to access the same disk and share files.
Since some services may take advantage of it, it's strongly suggested to load its drivers in the cluster.

To load the Azure file storage class, run:

```shell
kubectl apply -f system/common-azure-file-sc.yaml
```

In order to be able to use the Azure files storage class, both a ClusterRole and a ClusterRoleBinding need to be created.

To do so, from the *system* folder run:

```shell
kubectl apply -f system/common-azure-pvc-roles.yaml
```

##### Deploy Azure Storage PersistentVolumeClaim (PVCs)

PVCs are defined outside the helm-charts to avoid their deletion, while a chart gets removed for maintenance, or simply for human errors.
PVC definitions can be found in the *storage* folder. Each PVC is prefixed with the name of the chart that makes use of it, and should be created before installing the corresponding chart.

To create a PVC for a service, run

```shell
kubectl apply -f storage/SERVICE_NAME.yaml
```

PVCs can also be addded all at once, running

```shell
kubectl apply -f storage
```

#### Enable synchronization of Azure Keyvault secrets with Kubernetes secrets

It's strongly recommended to make Kubernetes retrieve secrets from the Azure Keyvault, instead of manually creating and editing secrets directly in Kubernetes. This approach is safer and allows an easier maintenance of the Kubernetes cluster.

The secrets synchronization and container injection is realized using [this component](https://akv2k8s.io/) (repo [here](https://github.com/SparebankenVest/azure-key-vault-to-kubernetes)).

Add the *spv-charts* repo and update the repo index:

```shell
helm repo add spv-charts http://charts.spvapi.no
helm repo update
```

Installation:

```shell
kubectl create namespace azurekeyvault

helm install \
    --set installCrd=false \
    --version 1.1.16 \
    --namespace azurekeyvault \
    key-vault-controller \
    spv-charts/azure-key-vault-controller

helm install \
    --version 1.1.21 \
    --namespace azurekeyvault \
    key-vault-env-injector \
    spv-charts/azure-key-vault-env-injector
```

Enable the automatic env variables injection for all containers in the default namespace:

```shell
kubectl apply -f system/common-azure-key-vault.yaml
```

Each chart that makes use of one or more secrets already contains an *azure-key-vault-secrets.yaml* file that creates

* A Kubernetes empty secret

* AzureKeyVaultSecret objects that trigger the pull of the secrets from the Azure Keyvault, synchronize the value with the local Kubernetes secret, and inject them as environment variables in the chart containers as needed

* Moreover, environment variables are imported in the *deployment.yaml* files with the value format `name-of-the-variable@azurekeyvault`

#### Deploy Velero for backups

[Velero](https://velero.io) and its [Azure plugin](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure) are used to backup Kubernetes resources, including taking volume snapshots, to an Azure blob storage account.

Make sure the service principal of Kubernetes (under registered applications in your [Azure Portal](https://portal.azure.com)) has a secret associated. Copy both the application client id and the secret. You'll need them in few minutes.

Create the *backups* namespace:

```shell
kubectl create namespace backups
```

Edit the template variables below (*YOUR_SERVICE_PRINCIPAL_CLIENT_ID* and *YOUR_SERVICE_PRINCIPAL_SECRET*) and create a secret in the your Azure Keyvault called *k8s-secrets-velero*:

```json
{
    "cloud": "AZURE_SUBSCRIPTION_ID=d30b533f-c88b-45ec-867e-8e321aa0b03f\nAZURE_TENANT_ID=f7f7d6c7-92de-488e-b37e-8963207c7b86\nAZURE_CLIENT_ID=YOUR_SERVICE_PRINCIPAL_CLIENT_ID\nAZURE_CLIENT_SECRET=YOUR_SERVICE_PRINCIPAL_CLIENT_SECRET\nAZURE_RESOURCE_GROUP=MC_devita-prod-rg_devita-prod-aks-k8s-01_westeurope\nAZURE_CLOUD_NAME=AzurePublicCloud"
}
```

Synchronize the Azure Keyvault secret with the local Kubernetes instance:

```shell
kubectl apply -f system/prod-velero-secrets.yaml
```

Install Velero:

```shell
# Add local Velero Helm repository
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm repo update

# Install/upgrade Velero
helm upgrade \
    -f system/prod-velero-custom.yaml \
    --namespace backups \
    --install \
    velero vmware-tanzu/velero
```

Enable backups for all namespaces:

```shell
kubectl apply -f system/common-velero-backup.yaml
```

List active schedules and backups with kubectl:

```shell
kubectl get schedules -n backups
kubectl get backups -n backups
```

Install Velero client and perform common tasks:

```shell
brew install velero

velero backup get
velero backup describe -n backups {backup-name}
velero backup logs -n backups {backup-name}
velero backup delete -n backups {backup-name}
```

#### Deploy the cert-manager    

The cert-manager is a Kubernetes component that takes care of adding and renewing TLS certificates for any virtual host specified in the ingress, through the integration with some providers (i.e. letsencrypt).

> **Warning:** If the first command generates a validation error, you should update the *kubectl* client.

To deploy the cert-manager run: 

```shell    
kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io   
helm repo update
helm install \
    --namespace cert-manager \
    --version v0.13.0 \
    cert-manager \
    jetstack/cert-manager
```

#### Apply cert-manager issuers

To integrate the cert-manager with the *letsencrypt certificate issuer*, run:

```shell
kubectl apply -f system/common-cert-manager-issuers.yaml
```

#### Deploy the ingress controller

The nginx ingress controller works as a reverse proxy, routing requests from the Internet to the applications living in the cluster. All applications DNS names are resolved to a single, public static IP address, that must be pre-provisioned on Azure.
Before proceeding, make sure you have allocated the public, static IP address using Terraform, and that the same address is reflected in the `nginx-ingress-custom.yaml` file.

We use a custom NGINX deployment to handle errors returned by NGINX, overriding the default error backend behavior. Double check the [NGINX configuration](system/common-nginx-custom-default-backend.yaml). Then deploy it:

```shell
kubectl apply -f system/common-nginx-custom-default-backend.yaml
```

Finally, install the NGINX ingress, running:

```shell
kubectl create namespace ingress

helm install \
    --namespace ingress \
    --version 1.29.3 \
    -f system/prod-nginx-ingress-custom.yaml \
    ingress \
    stable/nginx-ingress
```

### Deploy applications

Applications are packaged using helm charts.
Each helm chart configures one or more of the following services:

* Deployments

* Services and load balancers

* Nginx ingress rules (if any)

To list existing deployments

```shell
helm ls
```

To deploy a service

```shell
helm install [-f configs/CONFIG_NAME.yaml] [DEPLOYMENT_NAME] {NAME_OF_YOUR_CHART}
```

Where:

* CONFIG_NAME is optional and it's one of the configurations in the configs folder. By default, the *development* environment configuration is applied by default. So, for *dev* environments no configurations should be specified.

* DEPLOYMENT_NAME is optional, but strongly suggested. It represents an arbitrary name to give to the deployment (names can then be listed with `helm ls`, and used to reference charts in other helm commands)

* NAME_OF_YOUR_CHART is mandatory and corresponds to one of the folder names, each one representing the chart.

For example

```shell
helm install my-app my-app
```

Or, to deploy the same my-app, applying the production configuration

```shell
helm install -f configs/prod.yaml my-app my-app
```

To remove an existing service

```shell
helm delete [DEPLOYMENT_NAME]
```

For example:

```shell
helm delete my-app
```

## Redirect containers

Charts in the `redirects-*` directories are needed mainly to keep around legacy URLs.

They are deployed in the `redirects` namespace, for example:

```shell
helm install --create-namespace -n redirects redirects-design-italia-it redirects-design-italia-it
```

## How to contribute

Contributions are welcome. Feel free to [open issues](./issues) and submit a [pull request](./pulls) at any time.

## License

Copyright (c) 2020 Presidenza del Consiglio dei Ministri

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License along with this program.  If not, see <https://www.gnu.org/licenses/>.
