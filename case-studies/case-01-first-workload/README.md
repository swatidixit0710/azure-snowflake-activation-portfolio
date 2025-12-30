# Case Study 01 - First Workload Activation

## Customer Goal
The customer had data landing in Azure Blob Storage and needed
to quickly analyze it in Snowflake without exposing storage keys or over-engineering
the ingestion process.

The priority was **fast time-to-value** with **secure cloud-native access**.

---

## Architecture Overview
Azure Blob Storage serves as the landing zone for raw data.
Snowflake securely accesses the data using a **Storage Integration**
(no shared keys, no SAS tokens), then loads it using `COPY INTO`.

Key components:
- Azure Blob Storage (private container)
- Snowflake Storage Integration (Azure AD–based identity)
- External Stage
- Snowflake Warehouse (X-Small, auto-suspend)
- Target table (`BOOKS_RAW`)

---

## Implementation Steps (High Level)

### 1. Azure Setup
- Created a dedicated resource group for the project
- Created a storage account (Standard / LRS)
- Created a private container: `landing`
- Uploaded `books.csv`

### 2. Snowflake Setup
- Created a small warehouse (`ACTIVATION_WH`) with auto-suspend
- Created database, schema, and table (`BOOKS_RAW`)
- Created a **Storage Integration** for Azure
- Granted RBAC in Azure to the Snowflake Enterprise Application
- Created an external stage referencing the Azure container

### 3. Load & Validate
- Loaded data using `COPY INTO`
- Validated success with business queries

### 4. Troubleshooting
#### Issue: LIST @stage fails with authentication errors
##### Cause
-- Azure RBAC not granted to Snowflake service principal
-- Tenant consent not completed
##### Fix
-- Assign Storage Blob Data Reader role to Snowflake app
-- Open and accept AZURE_CONSENT_URL
-- Wait for RBAC propagation before retrying

#### Issue: COPY INTO loads 0 rows
##### Cause
-- File pattern mismatch
-- File format mismatch (header, delimiter, quoting)
##### Fix
-- Validate files with LIST @stage
-- Explicitly define file format
-- Use correct PATTERN



