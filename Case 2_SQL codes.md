### FitBit Fitness Tracker Data 

### Create table using pgAdmin 4

### minuteSleep_merged

```SQL
ALTER TABLE public."minuteSleep_merged_timestamp"
    ALTER COLUMN value TYPE integer USING value::integer
	
SELECT 
id,
sleep_start_day AS sleep_date,
COUNT(logid) AS number_naps,
SUM(EXTRACT(HOUR FROM time_sleeping)::int) AS total_hours_sleeping
FROM(
SELECT 
id,
logid,
MIN(date_trunc('day', date)) AS sleep_start_day,
MAX(date_trunc('day', date)) AS sleep_end_day,	
MIN(date) AS sleep_start,
MAX(date) AS sleep_end,
MAX(date) - MIN(date) AS time_sleeping
FROM public."minuteSleep_merged_timestamp"
WHERE value = 1
GROUP BY 
id,
logid) AS sleep_table
WHERE sleep_start_day = sleep_end_day
GROUP BY 
1,
2
ORDER BY
3 DESC;
```

### hourlyIntensities_merged

```SQL
SELECT 
	DISTINCT dow_number,
	part_of_week,
	day_of_week,
	time_of_day,
	PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_first_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.2) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_second_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.3) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_third_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.4) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_fourth_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_median,         --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.6) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_sixth_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.7) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_seventh_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) ,
	PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_eighth_decile,  --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 
	PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY totalintensity) AS total_intensity_ninth_decile    --(PARTITION BY dow_number, part_of_week, day_of_week, time_of_day) 

FROM
(
	SELECT
	id,
	totalintensity,
	EXTRACT(ISODOW FROM activityhour) AS dow_number,
	('{Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday}'::text[])[EXTRACT(ISODOW FROM activityhour)] AS day_of_week,
	(CASE 
	 WHEN EXTRACT(ISODOW FROM activityhour) IN (6, 7) THEN 'weekend'
	 WHEN EXTRACT(ISODOW FROM activityhour) NOT IN (6, 7) THEN 'weekday'
	 ELSE 'na'
	END) AS part_of_week,
	EXTRACT(HOUR FROM activityhour) AS exact_hour,
	(CASE
	WHEN EXTRACT(HOUR FROM activityhour) >= 6 AND EXTRACT(HOUR FROM activityhour) < 12 THEN 'morning'
    WHEN EXTRACT(HOUR FROM activityhour) >= 12 AND EXTRACT(HOUR FROM activityhour) < 18 THEN 'afternoon'
    WHEN EXTRACT(HOUR FROM activityhour) >= 18 AND EXTRACT(HOUR FROM activityhour) <= 21 THEN 'evening'
    ELSE 'others'
	END) AS time_of_day,
	sum(totalintensity) AS total_intensity,
    SUM(averageintensity) AS total_average_intensity,
    AVG(averageintensity) AS average_intensity,
    MAX(averageintensity) AS max_intensity,
    MIN(averageintensity) AS min_intensity

--(CASE WHEN EXTRACT(HOUR FROM activityhour) != 0 THEN (SUM(averageintensity)/EXTRACT(HOUR FROM activityhour))
--ELSE 0
--END) AS averageintensity_by---compare average not working tho
    FROM public."hourlyIntensities_merged"
    GROUP BY
    1,
    2,
    3,
    4,
    5,
6) AS user_dow_summary 
GROUP BY 1, 2, 3, 4 
````

### NTILE

```SQL
SELECT 
	DISTINCT dow_number,
    part_of_week,
	day_of_week,
	time_of_day,
	totalintensity,
	NTILE(10) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day ORDER BY totalintensity) AS intensity_deciles
	
FROM
(
	SELECT
	id,
	totalintensity,
	EXTRACT(ISODOW FROM activityhour) AS dow_number,
	('{Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday}'::text[])[EXTRACT(ISODOW FROM activityhour)] AS day_of_week,
	(CASE 
	 WHEN EXTRACT(ISODOW FROM activityhour) IN (6, 7) THEN 'weekend'
	 WHEN EXTRACT(ISODOW FROM activityhour) NOT IN (6, 7) THEN 'weekday'
	 ELSE 'na'
	END) AS part_of_week,
	EXTRACT(HOUR FROM activityhour) AS exact_hour,
	(CASE
	WHEN EXTRACT(HOUR FROM activityhour) >= 6 AND EXTRACT(HOUR FROM activityhour) < 12 THEN 'morning'
        WHEN EXTRACT(HOUR FROM activityhour) >= 12 AND EXTRACT(HOUR FROM activityhour) < 18 THEN 'afternoon'
        WHEN EXTRACT(HOUR FROM activityhour) >= 18 AND EXTRACT(HOUR FROM activityhour) <= 21 THEN 'evening'
        ELSE 'others'
	END) AS time_of_day,
	sum(totalintensity) AS total_intensity,
    SUM(averageintensity) AS total_average_intensity,
    AVG(averageintensity) AS average_intensity,
    MAX(averageintensity) AS max_intensity,
    MIN(averageintensity) AS min_intensity

--(CASE WHEN EXTRACT(HOUR FROM activityhour) != 0 THEN (SUM(averageintensity)/EXTRACT(HOUR FROM activityhour))
--ELSE 0
--END) AS averageintensity_by---compare average not working tho
    FROM public."hourlyIntensities_merged"
    GROUP BY
    1,
    2,
    3,
    4,
    5,
    6) AS user_dow_summary 
 GROUP BY 
   dow_number,
   part_of_week,
   day_of_week,
   time_of_day,
   totalintensity
```


