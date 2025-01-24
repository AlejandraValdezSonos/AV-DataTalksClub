# Homework: Week 1

## Question 1. Understanding docker first run
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash. What's the version of pip in the image?

Pull docker image 
``` bash
docker pull python:3.12.8
``` 
Run the container in interactive mode:
``` bash
docker run -it --entrypoint bash python:3.12.8
```
Confirm python version:
```bash
python --version
```
output: ``` Python 3.12.8```

Check the ```pip``` version inside the container I ran:
``` bash
pip --version
```
output: 
```pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)```

## Question 2: Understanding Docker Networking and Docker-compose
Given the following ```docker-compose.yaml```, what is the ```hostname``` and ```port``` that pgadmin should use to connect to the postgres database?
```yml
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
Answer: 
hostname: db
port: 5432

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
- 104,838;  199,013;  109,645;  27,688;  35,202 -- this is the answer?

``` SQl
SELECT
    SUM(CASE WHEN trip_distance <= 1 THEN 1 ELSE 0 END) AS "Up to 1 mile",
    SUM(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 ELSE 0 END) AS "Between 1 and 3 miles",
    SUM(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 ELSE 0 END) AS "Between 3 and 7 miles",
    SUM(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 ELSE 0 END) AS "Between 7 and 10 miles",
    SUM(CASE WHEN trip_distance > 10 THEN 1 ELSE 0 END) AS "Over 10 miles"
FROM
    green_taxi_trips
WHERE
    lpep_pickup_datetime >= '2019-10-01'
    AND lpep_pickup_datetime < '2019-11-01';
```


## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31 -- answer

```SQL
WITH daily_longest_trip AS (
    SELECT
        DATE(lpep_pickup_datetime) AS pickup_day,
        MAX(trip_distance) AS max_distance
    FROM
        green_taxi_trips
    GROUP BY
        DATE(lpep_pickup_datetime)
)
SELECT
    pickup_day,
    max_distance
FROM
    daily_longest_trip
ORDER BY
    max_distance DESC
LIMIT 1;
```


## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.
 
- East Harlem North, East Harlem South, Morningside Heights -- answer
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

```SQL
SELECT
    t."PULocationID",
    z."Zone" AS pickup_zone,
    SUM(t.total_amount) AS total_amount_sum
FROM
    green_taxi_trips t
JOIN
    taxi_zone_lookup z
ON
    t."PULocationID" = z."LocationID"
WHERE
    DATE(t.lpep_pickup_datetime) = '2019-10-18'
GROUP BY
    t."PULocationID", z."Zone"
HAVING
    SUM(t.total_amount) > 13000
ORDER BY
    total_amount_sum DESC;
```


## Question 6. Largest tip

For the passengers picked up in Ocrober 2019 in the zone
name "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport -- answer
- East Harlem North
- East Harlem South
```sql
SELECT
    d."Zone" AS dropoff_zone,
    MAX(t.tip_amount) AS largest_tip
FROM
    green_taxi_trips t
JOIN
    taxi_zone_lookup p ON t."PULocationID" = p."LocationID"
JOIN
    taxi_zone_lookup d ON t."DOLocationID" = d."LocationID"
WHERE
    DATE(t.lpep_pickup_datetime) >= '2019-10-01'
    AND DATE(t.lpep_pickup_datetime) < '2019-11-01'
    AND p."Zone" = 'East Harlem North'
GROUP BY
    d."Zone"
ORDER BY
    largest_tip DESC
LIMIT 1;
```
