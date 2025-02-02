# Kestra

You can use Kestra to:

- run workflows on-demand, event-driven or based on a regular schedule
- programmatically interact with any system or programming language
- orchestrate microservices, batch jobs, ad-hoc scripts (written in Python, R, Julia, Node.js, and more), SQL queries, data ingestion syncs, dbt or Spark jobs, or any other applications or processes

## Start kestra

```sh
docker run --pull=always --rm -it -p 8081:8080 --user=root -v /var/run/docker.sock:/var/run/docker.sock -v /tmp:/tmp kestra/kestra:latest server local
```

Open `localhost:8081` from the browser.

## Use docker compose

1. run `docker compose up` from terminal
1. open `localhost:8080`
