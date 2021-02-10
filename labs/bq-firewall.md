# BigQuery and GCP Firewall

[Link](https://cloud.google.com/blog/products/identity-security/google-cloud-firewall-rules-logging-how-and-why-you-should-use-it)

## Create a Firewall
```gcloud
gcloud compute firewall-rules create allow-enabled-firewall \
--network hub-vpc \
--allow tcp:22 \
--source-ranges 0.0.0.0/0 \
--enable-logging
```
## Create a BigQuery Dataset
```bash
bq mk --dataset <project id>:firewall_dataset
```

## Create the BigQuery sink to export the firewall rules logs:

```bash
gcloud logging sinks create sink_firewall bigquery.googleapis.com/projects/<project id>/datasets/firewall_dataset
--log-filter='logName:projects/<project id>/logs/compute.googleapis.com%2Ffirewall'
```

This will create a service account that needs to be granted `Data Editor` role on BigQuery dataset

```bash
# step 1
bq show —format=prettyjson <project id>:<dataset name>
> tmp_dataset_acls.json

# step 2
Edit tmp_dataset_acls.json file and add {role: “WRITER”, “userByEmail”: <service account email>} to the “access” list object.

# step 3
bq update --source tmp_dataset_acls <project id>:<dataset name>
```