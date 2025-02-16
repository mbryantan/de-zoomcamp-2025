# DLT Workshop: Homework


## Question 1: dlt Version

```python
import dlt
print("dlt version:", dlt.__version__)
```

ANSWER: dlt 1.6.1


## Question 2: Define & Run the Pipeline (NYC Taxi API)

```python
# my code is here
@dlt.resource(name="rides")
def ny_taxi():
    client = RESTClient(
        base_url = "https://us-central1-dlthub-analytics.cloudfunctions.net",
        paginator=PageNumberPaginator(
            base_page=1,
            total_path=None
        )
    )

    for page in client.paginate("data_engineering_zoomcamp_api"):
        yield page
```

ANSWER: 4


## Question 3: Explore the loaded data

```python
df = pipeline.dataset(dataset_type="default").rides.df()
df
```

ANSWER: 10000


## Question 4: Trip Duration Analysis

```python
with pipeline.sql_client() as client:
    res = client.execute_sql(
            """
            SELECT
            AVG(date_diff('minute', trip_pickup_date_time, trip_dropoff_date_time))
            FROM rides;
            """
        )
    # Prints column values of the first row
    print(res)
```

ANSWER: 12.3049
