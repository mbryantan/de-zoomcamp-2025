# Module 1 Homework: Docker & SQL

## Question 1. Understanding docker first run

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1

#### Answer

```bash
$ docker run -it --entrypoint bash python:3.12.8
$ pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```


## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

If there are more than one answers, select only one of them

#### Answer

```
db:5432
```

To test, open pgadmin in `http://localhost:8080/`,
- host name: db
- port: 5432
- maintenance database: postgres
- username: postgres
- password: postgres


##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.

#### Answer

Setup python environment using `poetry`:

poetry install guide: https://python-poetry.org/docs/

```bash
poetry new homework
poetry add pandas
poetry add sqlalchemy
poetry add psycopg2-binary
```

Tweak python script `ingest_data.py` from the course:
- rename `tpep` fields to `lpep` in the script.
- only parse `*_datetime` fields if they are actually present in the DF columns

```python
# load green_tripdata_2019-10.csv.gz to public.green_taxi_trips table
poetry run python ingest_data.py --user postgres --password postgres --host localhost --port 5433 --db ny_taxi --table_name green_taxi_trips --url https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz

# load taxi_zone_lookup.csv to public.taxi_zones
poetry run python ingest_data.py --user postgres --password postgres --host localhost --port 5433 --db ny_taxi --table_name taxi_zones --url https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Files are now loaded to `public.green_taxi_trips` and `public.taxi_zones` tables.

Data dictionaries are here for understanding the data:
- https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_green.pdf


## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- 104,802;  198,924;  109,603;  27,678;  35,189
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202

#### Answer

Query:
```sql
SELECT CASE WHEN trip_distance <= 1 THEN '0-1'
            WHEN trip_distance > 1 AND trip_distance <= 3 THEN '1-3'
            WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3-7'
            WHEN trip_distance > 7 AND trip_distance <= 10 THEN '7-10'
            ELSE 'over 10'
        END AS distance_grouping
     , COUNT(*)
  FROM green_taxi_trips
 WHERE lpep_pickup_datetime >= '2019-10-01'
   AND lpep_dropoff_datetime < '2019-11-01'
 GROUP BY 1;
```

Results:
```
104,802;  198,924;  109,603;  27,678;  35,189
```


## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

#### Answer

Query:
```sql
SELECT lpep_pickup_datetime::DATE
  FROM green_taxi_trips
 ORDER BY trip_distance DESC LIMIT 1
```

Results:
```
2019-10-31
```


## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.

- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

#### Answer

Query:
```sql
SELECT z."Zone"
     , SUM(t.total_amount) AS total_amount
     , COUNT(*) AS cnt
  FROM green_taxi_trips t
  LEFT JOIN taxi_zones z
    ON t."PULocationID" = z."LocationID"
 WHERE t.lpep_pickup_datetime >= '2019-10-18'
   AND t.lpep_pickup_datetime < '2019-10-19'
 GROUP BY 1 HAVING SUM(t.total_amount) > 13000
 ORDER BY 2 DESC;
```

Results:
```
East Harlem North, East Harlem South, Morningside Heights
```


## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone
named "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

#### Answer

Query:
```sql
SELECT z2."Zone", t.tip_amount
  FROM green_taxi_trips t
  LEFT JOIN taxi_zones z1
    ON t."PULocationID" = z1."LocationID"
  LEFT JOIN taxi_zones z2
    ON t."DOLocationID" = z2."LocationID"
 WHERE t.lpep_pickup_datetime >= '2019-10-01'
   AND t.lpep_pickup_datetime <  '2019-11-01'
   AND z1."Zone" = 'East Harlem North'
  ORDER BY t.tip_amount DESC LIMIT 1;
```

Result:
```
JFK Airport
```

## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform.
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.

#### Answer

##### Install Terraform

  Install using homebrew
  ```sh
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  brew update
  brew upgrade hashicorp/tap/terraform # upgrade to latest version
  ```

##### Create GCP project by folowing this [Initial Setup](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/01-docker-terraform/1_terraform_gcp/2_gcp_overview.md#initial-setup)

1. Create a new project: https://console.cloud.google.com
  - Project ID: `de-zoomcamp-2025-449011`
1. Create a service account: https://console.cloud.google.com/iam-admin/serviceaccounts
  - name it `terraform-runner`
  - Grant `Storage Admin` + `Storage Object Admin` + `BigQuery Admin` roles
1. Create a service account key in JSON format, it will be downloaded automatically.
1. Install Google Cloud CLI: https://cloud.google.com/sdk/docs/install-sdk
1. Set environment variable to point to the downloaded GCP service account key:
  ```bash
  export GOOGLE_APPLICATION_CREDENTIALS="/Users/<your-macos-name>/Downloads/<service-account-key.json>"

  # Refresh token/session, and verify authentication
  gcloud auth application-default login
  ```
1. Enable these APIs for your project:
  - [Identity and Access Management (IAM) API](https://console.cloud.google.com/apis/library/iam.googleapis.com)
  - [IAM Service Account Credentials API](https://console.cloud.google.com/apis/library/iamcredentials.googleapis.com)

##### Create resources using terraform
1. Setup `main.tf` and `variables.tf`. Update the project id and bucket name (make it unique).
1. Run `terraform init`
1. Run `terraform plan` and validate if the plan looks good. The expectation is that it will create a bigquery dataset and a cloud storage bucket.
1. Run `terraform apply`
  - Running `terraform apply -auto-approve` skips approval of plan before applying
1. Validate that the Bigquery dataset and GCS bucket have been created from the console:
  - https://console.cloud.google.com/storage/browser
  - https://console.cloud.google.com/bigquery
1. Cleanup the resources `terraform destroy`.


## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for:
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform

Answers:
- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-approve, terraform destroy
- terraform init, terraform apply -auto-approve, terraform destroy (HERE)
- terraform import, terraform apply -y, terraform rm

### Answer

```
terraform init, terraform apply -auto-approve, terraform destroy
```
