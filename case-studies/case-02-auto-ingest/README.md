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

## Snowpipe Definition (Conceptual)
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


