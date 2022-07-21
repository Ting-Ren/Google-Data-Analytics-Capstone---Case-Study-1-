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

### Update data

```SQL
UPDATE divvy_bikes
SET day_of_week = (EXTRACT(ISODOW FROM started_at));
```
