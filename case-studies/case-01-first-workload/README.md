# Case Study 01 — First workload: Azure Blob Storage → Snowflake

## Customer goal
Analyze book catalog data stored in Azure Blob Storage using Snowflake.

## Data
CSV file (`books.csv`) containing book metadata including authors, genres,
pricing, and publication year.

## What I built
- Azure Blob Storage private container as the landing zone
- Snowflake warehouse, database, schema, and table
- Secure access using Snowflake storage integration (no shared keys)
- External stage + COPY INTO
- Validation queries to confirm ingestion success

## Outcome
- Successfully loaded Azure Blob Storage data into Snowflake
- Validated row counts and business metrics
- Delivered first analytical value within minutes
- Established secure, production-ready access pattern


