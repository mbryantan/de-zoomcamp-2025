# Module 2 Homework: Workflow Orchestration using Kestra

### Initial Setup

1. Create GCP Project. We wil reuse GCP project from Module 1 (`de-zoomcamp-2025-449011`)
1. Prepare the JSON service account from Module 1 `terraform-runner`. This has the BQ Admin and Storage Admin GCP roles.
1. Open KV Store from Kestra: `http://localhost:8080/ui/namespaces/edit/zoomcamp/kv`
1. Ajust the flow `04_gcp_kv.yaml` and set the following Key-Values:
    - GCP_CREDS: Paste the service account key JSON from Module 1
    - GCP_PROJECT_ID: `de-zoomcamp-2025-449011`
    - GCP_LOCATION: `asia-southeast1`
    - GCP_BUCKET_NAME: `de-zoomcamp-2025-kestra-stg`
    - GCP_DATASET: `zoomcamp`

### Execute flow

1. Create and run the flow from `05_gcp_setup.yaml` to create the GCS bucket and BQ dataset.
1. Setup flow `06_gcp_taxi_scheduled.yaml` and schedule backfill for entire 2020 and 2021.

## Quiz Questions

1) Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size (i.e. the output file `yellow_tripdata_2020-12.csv` of the `extract` task)?

    Run `gsutil` command:

    `gsutil du -sh gs://de-zoomcamp-2025-kestra-stg/yellow_tripdata_2020-12.csv`

    ANSWER: `128.25 MiB` => `128.3 MB`

2) What is the rendered value of the variable `file` when the inputs `taxi` is set to `green`, `year` is set to `2020`, and `month` is set to `04` during execution?

    ANSWER: `green_tripdata_2020-04.csv`


3) How many rows are there for the `Yellow` Taxi data for all CSV files in the year 2020?

    QUERY:

    ```sql
    SELECT COUNT(*)
    FROM zoomcamp.yellow_tripdata
    WHERE filename LIKE '%_2020-%.csv';
    ```

    ANSWER: `24,648,499`

4) How many rows are there for the `Green` Taxi data for all CSV files in the year 2020?

    QUERY:

    ```sql
    SELECT COUNT(*)
    FROM zoomcamp.green_tripdata
    WHERE filename LIKE '%_2020-%.csv';
    ```

    ANSWER: `1,734,051`

5) How many rows are there for the `Yellow` Taxi data for the March 2021 CSV file?

    QUERY:

    ```sql
    SELECT filename, COUNT(*) AS cnt
    FROM zoomcamp.yellow_tripdata
    WHERE filename = 'yellow_tripdata_2021-03.csv'
    GROUP BY 1 ORDER BY 1;
    ```

    ANSWER: `1,925,152`


6) How would you configure the timezone to New York in a Schedule trigger?

    ANSWER: Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration
