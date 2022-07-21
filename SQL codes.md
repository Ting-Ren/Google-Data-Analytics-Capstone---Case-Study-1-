## Clean data

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
