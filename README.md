## 0 Setup 
Store the name of the sink, project and dataset in environment variables. The project you specify here, is the project where the dataset will live, with all the aggregated logs.
```bash
   export SINK_NAME=<YOUR_SINK_NAME>
   export PROJECT_ID=<YOU_PROJECT_NAME>
   export DATASET_NAME=<YOUR_DATASET_NAME>
```

Check if you have "the Logs Configuration Writer" role as individual.
 
## 1 Create the sink
To make an aggregated sink, you specifify either the --billing-acount, --folder or --organization ID. For this Cloud Identity or G-Suite has to be setup. See resources below. Add the --include-children, to see logs of all children resources. These are sub-folders, en all projects contained in this folder.
[Creating aggregreated reports](https://cloud.google.com/logging/docs/export/aggregated_exports)

```bash
    gcloud logging sinks create $SINK_NAME  \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/ --include-children \
    --billing-account=$BILLING_ID --log-filter="resource.type:Build"
```
Omiting these flags will default to your current project. Since I don't have folder level, organization level or billingID clearence, I will show you the same command, but on a project level. Note the --log-filter, which only select logs created by Cloud Builder.

```bash
    gcloud logging sinks create $SINK_NAME  \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/$DATASET_NAME --include-children \
    --log-filter="resource.type:Build"
```

## 2 Create a very simple query in BigQuery
Everything that happens in cloudbuild is now exported to the BigQuery Sink. 
To only see the status, i.e., start, done or error we add a simple filter.
```SQL
SELECT * FROM `<YOUR_TABLE_NAME>`
WHERE textPayLoad LIKE 'ERROR'
   OR textPayLoad LIKE '%starting build%'
   OR textPayLoad Like 'DONE'
ORDER BY timeStamp DESC 
```
Now you can create a view store to store the query, have it updated periodically, and feed in a simple dashboard

## Some resources
Resources: 
[Creating aggregreated reports](https://cloud.google.com/logging/docs/export/aggregated_exports)
[Setup export sink with gcloud](https://cloud.google.com/sdk/gcloud/reference/beta/logging/sinks/create)
