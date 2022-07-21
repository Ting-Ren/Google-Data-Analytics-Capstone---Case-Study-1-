### Divvy_Exercise_Full_Y ear_Analysis ###
# This analysis is based on the Divvy case study "'Sophisticated, Clear , and Polished’: Divvy and Data Visualization" written by Kevin Hartman (found here:
https://artscience.blog/home/divvy-dataviz-case-study). 

### Create full year table

```SQL
CREATE TABLE "Divvy_Trips_Full_Year" (
	"ride_id"	TEXT,
	"started_at"	TEXT,
	"ended_at"	TEXT,
	"rideable_type"	TEXT,
	"start_station_id"	TEXT,
	"start_station_name"	TEXT,
	"end_station_id"	TEXT,
	"end_station_name"	TEXT,
	"member_casual"	TEXT
)

INSERT INTO Divvy_Trips_Full_Year
SELECT *
FROM
(
SELECT * 
FROM Divvy_Trips_2019_Q2
UNION ALL 
 SELECT * 
FROM Divvy_Trips_2019_Q3
UNION ALL
SELECT * 
FROM Divvy_Trips_2019_Q4
UNION ALL
SELECT * 
FROM Divvy_Trips_2020_Q1);
```


### Clean data

```SQL
SELECT  member_casual, COUNT(DISTINCT ride_id)
FROM Divvy_Trips_Full_Year
GROUP BY member_casual

UPDATE Divvy_Trips_Full_Year
SET
member_casual= replace(member_casual, 'Subscriber', 'member') 

UPDATE Divvy_Trips_Full_Year
SET
member_casual= replace(member_casual, 'Customer', 'casual') 
```

### Update date

```SQL
UPDATE divvy_bikes
SET day_of_week = (EXTRACT(ISODOW FROM started_at));
```

### Descriptive analysis on ride_length

```SQL
SELECT AVG(r.ride_length), MAX(r.ride_length), MIN(r.ride_length),
PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER by r.ride_length) AS median
FROM 
(
SELECT EXTRACT(EPOCH FROM (ended_at - started_at)) AS ride_length
FROM divvy_bikes
WHERE start_station_name != 'HQ QR'
AND EXTRACT (EPOCH FROM (ended_at - started_at)) > 0
ORDER BY EXTRACT (EPOCH FROM (ended_at - started_at)) ASC
) r
```

### Compare members and casual users & day_of_week 

```SQL
--number_of_rides

SELECT member_casual, day_of_week, COUNT(ride_id) AS numbr_of_ride, AVG(r.ride_length)
FROM 
(
SELECT member_casual, day_of_week, ride_id, EXTRACT(EPOCH FROM (ended_at - started_at)) AS ride_length
FROM divvy_bikes
WHERE start_station_name != 'HQ QR'
AND EXTRACT (EPOCH FROM (ended_at - started_at)) > 0
ORDER BY EXTRACT (EPOCH FROM (ended_at - started_at)) ASC
) r
GROUP BY member_casual, day_of_week
ORDER BY day_of_week
```
