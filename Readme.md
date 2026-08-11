# Guide to Setting Up BigLake Connection in Google Cloud

As a best practice in data lake architecture, BigLake requires a Connection (Cloud Resource) so BigQuery can delegate data read access to Google Cloud Storage (GCS) securely using a Service Account, without exposing individual user credentials.

## Step 1: Creating a BigLake Connection

1. Open the **BigQuery Console**.
2. In the Explorer panel, click **+ ADD** -> choose **Connections to external data sources**.
3. In the **Connection type** dropdown, select:
   **Agent Platform remote models, remote functions, Lakehouse and Spanner (Cloud Resource)**
4. Change the **Location type** to **Region**.
5. Select your primary operational region (e.g., `asia-southeast2` for Jakarta) to optimize query latency and avoid data egress costs.
6. Name the connection (example: `biglake-data-connectionconn`).
7. Click **Create Connection**.
8. Open the newly created connection in the Explorer panel, then copy the generated **Service Account ID** (ending with `@gcp-sa-bigquery-condel.iam.gserviceaccount.com`).

## Step 2: IAM (Identity and Access Management) Configuration

The Service Account associated with the BigLake connection must be granted permissions to read objects in the target GCS bucket.

1. Open the **Cloud Storage > Buckets** page.
2. Navigate to your target bucket (example: `gs://biglake_data`).
3. Go to the **Permissions** tab and click **+ GRANT ACCESS**.
4. *Paste* the **Service Account ID** into the *New principals* field.
5. Select the role: **Cloud Storage** -> **Storage Object Viewer**.
6. Click **Save**.
   *(Note: IAM propagation typically takes about 1-2 minutes before BigQuery can utilize the new permissions).*

---

## ⚠️ Common Setup Troubleshooting

### Error 1: Invalid Connection Name Format
**Error Message:** 
`Connection name should conform to the pattern: projects/{project_id=*}/locations/{location_id=*}/connections/{connection_id=*}`

**Cause:** 
In the BigQuery DDL query, referencing the Connection by its name alone is insufficient. The BigQuery engine requires the region information.

**Solution:**
Add the region (location) prefix in front of the connection name in the `WITH CONNECTION` clause.
*Incorrect:* `WITH CONNECTION 'biglake-data-connectionconn'`
*Correct:* `WITH CONNECTION 'asia-southeast2.biglake-data-connectionconn'`

### Error 2: Permission Denied (Globbing File Pattern)
**Error Message:** 
`Access Denied: Permission denied while globbing file pattern. <service-account> does not have storage.objects.list access...`

**Cause:** 
The BigLake connection is invoked in the DDL, but its Service Account has not been granted permission (or the permission hasn't propagated yet) to read the contents of the GCS Bucket.

**Solution:**
Follow **Step 2** above to assign the **Storage Object Viewer** role at the bucket level. Wait 1-2 minutes before running the DDL again.
