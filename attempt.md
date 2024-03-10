# Question 1
Create the materialized view trip_stats.
```sql
CREATE MATERIALIZED VIEW trip_stats AS
    SELECT
        tpu.Zone AS pick_up_zone,
        tdo.Zone AS drop_off_zone,
        MAX((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS max_trip_time,
        MIN((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS min_trip_time,
        AVG((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS avg_trip_time
    FROM
        trip_data t
        JOIN taxi_zone tpu
        ON t.PULocationID = tpu.location_id
        JOIN taxi_zone tdo
        ON t.DOLocationID = tdo.location_id
    GROUP BY pick_up_zone, drop_off_zone;
```
Answer the question of the pair of taxi zones with the highest average trip time.
```sql
SELECT
    pick_up_zone,
    drop_off_zone,
    avg_trip_time
FROM
    trip_stats
ORDER BY avg_trip_time DESC
LIMIT 1;
```

# Question 2
Create the modified materialized view trip_stats.
```sql
CREATE MATERIALIZED VIEW trip_stats_2 AS
    SELECT
        tpu.Zone AS pick_up_zone,
        tdo.Zone AS drop_off_zone,
        MAX((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS max_trip_time,
        MIN((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS min_trip_time,
        AVG((EXTRACT(EPOCH FROM tpep_dropoff_datetime) - EXTRACT(EPOCH FROM tpep_pickup_datetime)) / 60) AS avg_trip_time,
        COUNT(*) AS num_trips
    FROM
        trip_data t
        JOIN taxi_zone tpu
        ON t.PULocationID = tpu.location_id
        JOIN taxi_zone tdo
        ON t.DOLocationID = tdo.location_id
    GROUP BY pick_up_zone, drop_off_zone;
```
Answer the question of the number of taxi trips.
```sql
SELECT
    SUM(num_trips)
FROM
    trip_stats_2
WHERE
    pick_up_zone = 'Yorkville East'
    AND drop_off_zone = 'Steinway';
```

# Question 3
Create the materialized view trip_stats_3 with dynamic filter condition for the latest pickup time.
```sql
CREATE MATERIALIZED VIEW trip_stats_3 AS
    WITH v AS (
        SELECT
            MAX(tpep_pickup_datetime) - INTERVAL '17 hours' AS earliest_pickup_time
        FROM
            trip_data
    )
    SELECT
        tpu.Zone AS pick_up_zone,
        COUNT(*) AS trip_count
    FROM
        v,
        trip_data t
        JOIN taxi_zone tpu
        ON t.PULocationID = tpu.location_id
    WHERE
        t.tpep_pickup_datetime >= v.earliest_pickup_time
    GROUP BY
        pick_up_zone;
```
Getting the answer for the 3 busiest zones in terms of the number of pickups
```sql
SELECT
    *
FROM
    trip_stats_3
ORDER BY trip_count DESC
LIMIT 3;
```