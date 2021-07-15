# Dashboard app

The dashboard application reads data from some CSV and populates a MongoDB. Data from the MongoDB are then read by [Metabase](https://metabase.com/), which can be used to display statistics and show some charts.

## Requirements

The following steps should be performed before being able to install metabase and the local chart in this folder.

### Storage and PVCs

The chart makes use of some Kubernetes PersistentVolumeClaims (PVCs) to maintain data persistence (i.e. data in MongoDB, Postgres). PVCs should be independent from the chart lifecycle, so that they're not removed every time the chart gets removed. For this reason, their configuration files are stored in a separate directory, called storage.
Make sure to apply with *kubectl* the chart corresponding PVC first, before continuing.

### Storage creation to support SPID data ingestion

SPID related CSV files are periodically uploaded manually by users of the Team. To enable such functionality it's important to first create a special storage in Azure where the CSV can be uploaded.

To create the storage, some [live Terraform modules](https://github.com/teamdigitale/dpt-services-infra-tf-live) should be applied.

These modules are, in order:

* [storage_account_dashboard_spid](https://github.com/teamdigitale/dpt-services-infra-tf-live/tree/dev/westeurope/storage_account_dashboard_spid)

* [storage_account_dashboard_spid_share](https://github.com/teamdigitale/dpt-services-infra-tf-live/tree/dev/westeurope/storage_account_dashboard_spid_share).

Once the storage share has been created in Azure, create a secret in Kubernetes to let the SPID container connect to it as needed:

```shell
kubectl create secret generic dashboard-secret-spid --from-literal=azurestorageaccountname=YOUR_SPID_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=YOUR_SPID_STORAGE_ACCOUNT_KEY --namespace=dashboards
```

Both the storage account name and the storage account key should be retrieved from the Azure storage account section in the GUI.

### Create secrets in Azure Keyvault and sync it in Kubernetes

Create in the Azure Keyvault a secret named *k8s-secrets-dashboard-metabase*.

The value of the secret should be in the following format:

```json
{
  "postgresql-username": "YOUR_POSTGRESQL_USERNAME",
  "postgresql-password": "YOUR_POSTGRESQL_PASSWORD",
  "postgresql-replication-password": "YOUR_POSTGRESQL_REPLICATOIN_PASSWORD",
  "mongodb-root-password": "YOUR_MONGODB_ROOT_PASSWORD",
  "mongodb-password": "YOUR_MONGODB_PASSWORD",
  "forum_api_key": "YOUR_FORUM_API_KEY",
  "token_github": "YOUR_GITHUB_TOKEN",
  "token_slack": "YOUR_SLACK_TOKEN",
  "google_project_id": "YOUR_GOOGLE_PROJECT_ID",
  "google_private_id": "YOUR_GOOGLE_PRIVATE_ID",
  "google_private_key": "YOUR_GOOGLE_PRIVATE_KEY",
  "google_client_email": "YOUR_GOOGLE_CLIENT_EMAIL",
  "google_client_id": "YOUR_GOOGLE_CLIENT_ID",
  "google_client_x509_cert_url": "YOUR_GOOGLE_X509_CLIENT_CERT_URL",
  "google_wpid": "YOUR_GOOGLE_WP_ID"
}
```

### Add basic auth as an extra-protection for the Metabase admin page

```shell
# Create the basic-auth file
htpasswd -c auth THE-USER-YOU-WANT

# Add it as a secret in Kubernetes
kubectl create secret generic admin-basic-auth --from-file=auth
```

### Install the charts and its dependencies

The chart will install the cronjob to periodically ingest new data. It will also install the following tools, using the official charts:

* Metabase

* Postgres (used to support Metabase)

* MongoDB (where data are saved and read by Metabase)

In order to upgrade all components (cronjob included) it is necessary to uninstall the entire chart as follow

```shell
helm uninstall -n dashboards dashboard-metabase
```

Download dependencies if they not yet exist in dashboard-metabase/chart/charts and then install
```shell
cd dashboard-metabase/chart && helm dep update && cd ../../

helm install \
  --namespace dashboards \
  dashboard-metabase \
  dashboard-metabase/chart
```
## Metabase and its databases

In this specific configuration, Metabase uses two databases to provide different functionalities:

* **Postgres** keeps Metabase configurations

* **MongoDB** hosts data files

The databases are automatically installed by the main dashboard chart as dependencies, using the official database charts. You can see more in the installation section below.

### Import data from old Postgres deployments

To export data from Postgres, enter the old container. Then, use pg_dump to export your data. Finally, copy the file to your local machine.

```shell
kubectl exec -n dashboards -it dashboard-postgresql-bdfaggafdd-retrw /bin/bash

pg_dump -U postgres -W -F t metabase > /tmp/postgres.tar
exit

kubectl cp -n dashboards dashboard-postgresql-bdfaggafdd-retrw:/tmp/postgres.tar /tmp/postgres.tar
```

To restore the exported data, copy the database backup file to the container. Then, enter in the new Postgres container and make sure no old data are present, dropping the schema and re-creating it. Finally, restore the data with the *pg_restore* tool.

```shell
kubectl cp -n dashboards postgres.tar dashboard-postgresql-6fd55f699d-srqxl:/tmp/postgres.tar

kubectl exec -n dashboards -it dashboard-postgresql-6fd55f699d-srqxl /bin/bash

psql -U metabase -d metabase
DROP SCHEMA IF EXISTS public CASCADE;
CREATE SCHEMA public;
\q

pg_restore -U metabase -d metabase /tmp/postgres.tar

exit
```

### Import data from old MongoDB deployments

To export data from MongoDB, enter the old MongoDB container. Then, tar the file to make it smaller and copy it to your computer

```shell
kubectl exec -n dashboards -it dashboard-mongodb-7sadfafad-abvcx /bin/bash

cd /tmp
mongodump -u 'teamdigitale' -p '<mongodb password>' --authenticationDatabase "monitor_mdb" -d monitor_mdb  -o .
tar -czf mongodb.tar.gz monitor_mdb
exit

kubectl cp -n dashboards dashboard-mongodb-7sadfafad-abvcx:/tmp/mongodb.tar.gz ~/tmp/mongodb.tar.gz
```

Let's start the export. Copy the MongoDB dump file to the new container using *kubectl cp*:

```shell
kubectl cp -n dashboards /tmp/mongodb.tar.gz dashboard-mongodb-6fd55f699d-srqxl:/tmp/mongodb.tar.gz
```

Enter again the MongoDB container, untar the file and restore the data using the *mongorestore* tool:

```shell
kubectl exec -n dashboards -it dashboard-mongodb-6fd55f699d-srqxl /bin/bash
cd /tmp
tar -xvzf mongodb.tar.gz
mongorestore --host localhost:27017 --username teamdigitale --password '<your-mongodb-password>' --authenticationDatabase monitor_mdb --db monitor_mdb monitor_mdb
```
