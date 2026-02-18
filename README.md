# Project 1 – Incremental Spark ETL with Correctness and Performance Evidence

## Overview
This project implements an incremental ETL pipeline using Apache Spark to process NYC Taxi trip records stored as Parquet files.
The job processes only new incoming files, applies data cleaning and deduplication rules, enriches trips with zone information, and writes a cumulative enriched output dataset.
The pipeline is idempotent, meaning it can be safely re-run without duplicating data.

---

## Dataset
- NYC Taxi and Limousine Commission (TLC) trip records, provided as monthly Parquet files
- taxi_zone_lookup.parquet: static lookup table mapping LocationID to zone names

Input files location:
data/inbox/

Lookup table location:
data/taxi_zone_lookup.parquet

---

## Architecture and Incremental Ingestion

### Inbox / Outbox Pattern
- Inbox (data/inbox/)
  Contains raw input Parquet files (monthly trip data). These files are never modified.

- Outbox (data/outbox/trips_enriched.parquet)
  Contains the cumulative, cleaned, deduplicated, and enriched dataset.

### Manifest (State Management)
A state file is maintained at:
state/manifest.json

The manifest records which input files have already been processed.
At minimum, it stores the filename and processing timestamp.

Example manifest entry:
filename: yellow_tripdata_2025-01.parquet  
processed_at: 2026-02-15T19:10:00Z

### Incremental Logic
1. List all Parquet files in data/inbox/
2. Compare them with filenames stored in the manifest
3. Process only new files
4. Append processed data to the outbox
5. Update the manifest

Re-running the job when no new files are present results in:
Nothing new to process.

---

## Data Cleaning
The following data cleaning rules were applied:

1. Remove trips with passenger_count <= 0
2. Remove trips with trip_distance <= 0
3. Remove trips where dropoff_datetime <= pickup_datetime

These rules remove logically invalid taxi trips caused by sensor or data entry errors.

### Examples of Invalid Rows
- Trips with zero passengers
- Trips with non-positive distance
- Trips with invalid pickup or dropoff timestamps

---

## Deduplication
To avoid duplicate trips, records are deduplicated using the composite key:

VendorID  
pickup_datetime  
dropoff_datetime  
PULocationID  
DOLocationID  

Deduplication is applied to newly ingested data before writing to the output, ensuring correctness and idempotency.

---

## Data Enrichment
Trip records are enriched using taxi_zone_lookup.parquet:

- pickup_zone is added using PULocationID
- dropoff_zone is added using DOLocationID

Two left joins are applied to preserve all trips even if a zone mapping is missing.

To improve performance, the lookup table is joined using broadcast joins.

---

## Derived and Metadata Fields
The following additional fields are created:

- trip_duration_minutes – derived from pickup and dropoff timestamps
- pickup_date – date extracted from pickup timestamp
- source_file – original input file that produced the record
- ingested_at – timestamp when the record was processed

---

## Output
- Output is written to:
data/outbox/trips_enriched.parquet

- The dataset grows incrementally as new files arrive
- Previously processed data is preserved
- No duplicates are created when re-running the job

---

## Correctness Evidence

Stage | Row Count
----- | ---------
Input (new files) | XXXX
After cleaning | XXXX
After deduplication | XXXX
Final output (cumulative) | XXXX

The counts demonstrate that invalid and duplicate records are removed and that the output accumulates correctly across runs.

---

## Performance Evidence
Performance was evaluated using the Spark Web UI.

Metrics collected include:
- Total job execution time
- Stage execution time
- Shuffle read/write during join operations

### Optimizations Applied
1. Broadcast joins for the zone lookup table
2. Reduced number of shuffle partitions
3. Append-only writes to avoid expensive global rewrites

Spark UI screenshots showing job runtime and shuffle metrics are included.

---

## Idempotency
The job is idempotent due to:
- File-level tracking in the manifest
- Processing only new input files
- Deduplication logic based on a composite key

Re-running the job without new input files does not modify the output.

---

## Custom Scenario
The custom scenario provided by the teaching staff was implemented as required.
Details and implementation can be found in the corresponding notebook cells and repository commits.

---

## How to Run the Job
1. Place new Parquet files in data/inbox/
2. Start the Spark environment (Docker + Jupyter)
3. Run the incremental ETL job cell in the notebook
4. Verify that:
   - New data is appended to the outbox
   - The manifest is updated
   - Re-running without new files produces no changes

---

## Technologies Used
- Apache Spark (PySpark)
- Parquet
- Docker
- Jupyter Notebook
- Python
