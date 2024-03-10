Question 1:
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
Answer the question of the 3 busiest zones by the number of pickup trips.
```sql
SELECT
    *
FROM
    trip_stats_3
ORDER BY
    trip_count DESC
LIMIT 3;
```