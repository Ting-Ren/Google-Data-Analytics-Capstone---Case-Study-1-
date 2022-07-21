### R code for basic visualization

View plots via Kaggle notebook https://www.kaggle.com/code/tingrentr/google-data-analytics-capstone-case-study 

```R
library(tidyverse)
daily_activity<- read.csv("dailyActivity_merged.csv")
head(daily_activity)
sleep_day<- read.csv("sleepDay_merged.csv")

colnames(daily_activity)
colnames(sleep_day)

n_distinct(daily_activity$Id)
n_distinct(sleep_day$Id)

daily_activity %>% 
select(TotalSteps,
TotalDistance,
SedentaryMinutes) %>% 
summary()

sleep_day %>% 
select(TotalSleepRecords,
TotalMinutesAsleep,
TotalTimeInBed) %>% 
summary()


ggplot(data = daily_activity, 
aes(x = TotalSteps, y = SedentaryMinutes))+
geom_point()
## result: more sedentarymiutes, less steps

ggplot(data = sleep_day, 
aes(x = TotalMinutesAsleep, y = TotalTimeInBed)) +
geom_point()
## result: timeinbed>minutesasleep, indidating not effective sleeping or insomnia

combined_data<- merge(sleep_day, daily_activity, by = "Id")
n_distinct(combined_data$Id)

ggplot(data = combined_data, 
aes(x = TotalMinutesAsleep, y = TotalSteps)) +
         geom_point(na.rm = TRUE)
## result: no clear relationship
```
