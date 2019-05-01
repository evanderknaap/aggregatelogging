## 0 Setup 
Store the name of the sink, project and dataset in environment variables.
```bash
   export SINK_NAME=<YOUR_SINK_NAME>
   export PROJECT_ID=<YOU_PROJECT_NAME>
   export DATASET_NAME=<YOUR_DATASET_NAME>
```

Check if you have "the Logs Configuration Writer" role as individual.
 
## 1 Create the sink
To make an aggregated sink, you specifify either the --billing-acount, --folder or --organization flag. For this to work, one has to have an organization. If you don;t have this yet, you can setup Cloud Identity or use your G-Suite domain. See resources below. 

Add the --include-children, to see logs of all children resources. This means that all project in all sub-folders are included in the logs.
[Creating aggregreated reports](https://cloud.google.com/logging/docs/export/aggregated_exports)

```bash
    gcloud logging sinks create $SINK_NAME  \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/ --include-children \
    --billing-account=$BILLING_ID --log-filter="resource.type:Build"
```
If you omit the --billing-account, --folder and --organization flas, the sink will default to the scope of the current project. I don't have folder level clearence, hence I will show you the same command, but on a project level only.

```bash
    gcloud logging sinks create $SINK_NAME  \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/$DATASET_NAME --include-children \
    --log-filter="resource.type:Build"
```

## 2 Create a simple query in BigQuery
Everything that happens in Cloud Build is now exported to our new BigQuery Sink. 
To only see the status, i.e., START, DONE or SUCCES - we simply filter on these statements in the log payload.
```SQL
SELECT * FROM `<YOUR_TABLE_NAME>`
WHERE textPayLoad LIKE 'ERROR'
   OR textPayLoad LIKE '%starting build%'
   OR textPayLoad Like 'DONE'
ORDER BY timeStamp DESC 
```

Or slightly more cleaned up ...
```SQL
SELECT
  resource.labels.project_id AS PROJECT,
  resource.labels.build_id AS BUILD_ID,
  CASE
    WHEN textPayLoad LIKE 'ERROR' THEN "FAILED"
    WHEN textPayLoad LIKE '%starting build%' THEN "STARTED"
    ELSE "SUCCESS"
  END AS STATUS,
  timeStamp 
FROM
  `<YOUR_TABLE_NAME>`
WHERE
  textPayLoad LIKE 'ERROR'
  OR textPayLoad LIKE '%starting build%'
  OR textPayLoad LIKE 'DONE'
ORDER BY BUILD_ID, timeStamp DESC
```

Now you can save the query and save view which feeds a simple dashboard. In the BigQuery UI, click "save view" and give it a name. 
Go to [datastudio.google.com](www.http://datastudio.google.com), and create a report with the table view as source. 


## Some resources
Resources: 
[Creating aggregreated reports](https://cloud.google.com/logging/docs/export/aggregated_exports)
[Setup export sink with gcloud](https://cloud.google.com/sdk/gcloud/reference/beta/logging/sinks/create)
