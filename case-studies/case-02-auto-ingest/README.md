# Case Study 02 — Automated Ingestion with Snowpipe (Azure)

## Customer Scenario
After successfully validating an initial workload using manual ingestion,
the customer required an automated ingestion solution as new files arrived
continuously in Azure Blob Storage.

The goal was to reduce operational overhead while maintaining secure,
cloud-native access patterns.

---

## Why Snowpipe
Snowpipe is Snowflake’s serverless ingestion service designed for:
- Event-driven, near-real-time ingestion
- Minimal operational effort
- Automatic scaling
- Cost efficiency for incremental data

It is the recommended approach once ingestion becomes recurring.

---

## Architecture Overview
- Azure Blob Storage (private container `landing`)
- Azure Event Grid (BlobCreated events)
- Azure Storage Queue (`snowpipe-books-queue`)
- Snowflake Notification Integration
- Snowflake Snowpipe (`books_snowpipe`)
- Target table: `BOOKS_RAW`

---

## Implementation Summary
1. Registered `Microsoft.EventGrid` provider at the subscription level
2. Created Azure Storage Queue for Snowpipe notifications
3. Created Event Grid system topic and event subscription
4. Created Snowflake Notification Integration for Azure Storage Queue
5. Granted Azure RBAC permissions to Snowflake Enterprise Application
6. Created Snowpipe with `AUTO_INGEST = TRUE`
7. Validated automatic ingestion by uploading new files

---

## Snowpipe Definition
```sql
CREATE PIPE books_snowpipe
  AUTO_INGEST = TRUE
  INTEGRATION = 'BOOKS_NOTIF_INT'
AS
COPY INTO BOOKS_RAW
FROM @azure_books_stage
PATTERN = '.*\\.csv';
```
---

### 4. Troubleshooting
### Issue: Error: `Integration cannot be null for AZURE`
### Cause
- For Azure-based Snowpipe, a **Notification Integration** is mandatory.  
- A Storage Integration alone only allows data access, not event-based ingestion.
### Fix
- Created a Notification Integration using `AZURE_STORAGE_QUEUE`
- Referenced it explicitly in the pipe definition:
```sql
INTEGRATION = 'BOOKS_NOTIF_INT'
```

### Issue: Snowflake application not visible in Azure IAM
### Cause
- Azure tenant consent had not been granted yet
- The Enterprise Application had not been materialized in the tenant
### Fix
- Opened and accepted the AZURE_CONSENT_URL from Snowflake
- Waited several minutes for Azure propagation
- Located the app under Microsoft Entra ID → Enterprise Applications
- Assigned RBAC after the app appeared

### Issue: Error authenticating with Azure when accessing stage or Snowpipe
### Cause
- Missing or incorrect Azure RBAC permissions 
### Fix
- Assigned Storage Blob Data Reader to the Snowflake app on the Storage Account
- Assigned Storage Queue Data Contributor to the Snowflake app on the Storage Queue
- Waited for RBAC propagation
- Verified access using:
```sql
LIST @azure_books_stage;
```

### Issue: Event Grid system topic creation fails
### Cause
- The Event Grid resource provider was not registered for the subscription
### Fix
- Azure Portal → Subscriptions → Resource providers
- Registered Microsoft.EventGrid
- Retried Event Grid configuration

### Issue: Files uploaded but Snowpipe does not trigger
### Cause
- Event Grid subject filter mismatch
- Missing queue permissions
- Files existed before Snowpipe creation
### Fix
- Verified Event Grid subject filter
- Confirmed queue permissions for the Snowflake app
- Loaded existing files using:
```sql
ALTER PIPE books_snowpipe REFRESH;
```




