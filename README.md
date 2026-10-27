# Databricks Taxi Lakehouse Pipeline

I built this project to get hands-on with the Medallion Architecture on Databricks. It takes NYC Yellow Taxi trip data (Jan 2023) from raw Parquet files and processes it through Bronze, Silver, and Gold layers, ending in a small star schema with a fact table and dimension tables. Everything runs on Databricks Free Edition using PySpark and Delta Lake, with Unity Catalog managing the tables.

The pipeline has three notebooks:

- `01_bronze_ingest.py` — reads the raw Parquet file from a UC Volume and writes it to `workspace.taxi.bronze_trips` as-is.
- `02_silver_clean.py` — deduplicates trips, casts columns to proper types, drops invalid rows (negative fares, zero distance), and writes `silver_trips`.
- `03_gold_star_schema.py` — builds `fact_trips`, `dim_vendors`, and `dim_locations` (from the TLC zone lookup CSV). Also checks that every pickup location ID in the fact table exists in `dim_locations`.

A 3-task Databricks Job runs them in order: Bronze → Silver → Gold.

## How to run it

1. Run `sql/00_setup_schema_volume.sql` to create the `workspace.taxi` schema and `raw_data` volume.
2. Upload `yellow_tripdata_2023-01.parquet` and `taxi_zone_lookup.csv` to that volume.
3. Run the Databricks Job (or run the three notebooks manually in order).
4. Check results with the queries in `sql/99_validation_queries.sql`.

Both data files are public — links are in the `sql/00_setup_schema_volume.sql` comments.

## Notes

- Databricks Free Edition is serverless-only and blocks outbound internet, so I uploaded the source files to a UC Volume instead of fetching them from within a notebook.
- All transformations use `saveAsTable` so the tables are governed by Unity Catalog. Lineage is visible in the Catalog UI.
- On a paid workspace, the same notebooks work — just point them at an ADLS Gen2 external location and add Auto Loader for incremental ingestion. The transformation logic doesn't change.

## Dataset

NYC TLC Yellow Taxi Trip Data:
- Trips: https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet
- Zone lookup: https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
