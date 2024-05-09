# BellaBeat_project
This case study from the Google Analytics course offers me a chance to demonstrate my technical skills in SQL, Tableau, and R programming.
# Introduction
Bellabeat, established in 2013, specializes in the development of sophisticated, health-focused smart products for women. The company aims to inspire and empower women by providing valuable insights into their health and habits, quickly becoming a leader in female-centric wellness technology.

Urška Sršen, Co-founder and Chief Creative Officer, advocates for the strategic analysis of external consumer data, such as FitBit fitness tracker usage. She believes this approach will reveal further opportunities for Bellabeat's growth and development.
# Objective
The objective is to analyze data from the FitBit Fitness Tracker App to uncover insights into user behaviors. These insights will inform the marketing strategies that aim to solidify Bellabeat's position as a global tech industry leader.

# Stakeholders
Primary Stakeholders:
- Urška Sršen: Co-founder and Chief Creative Officer of Bellabeat.
- Sando Mur: Co-founder of Bellabeat and a key mathematician on the executive team.
  
Secondary Stakeholders:
- Bellabeat Marketing Analytics Team: This team comprises data analysts dedicated to collecting, analyzing, and reporting data to refine Bellabeat’s marketing approaches.

# Data Preparation
Data Generation and Collection:
The data for this analysis originates from [Kaggle](https://www.kaggle.com/datasets/arashnic/fitbit). For detailed dataset documentation, please refer to [this link](https://www.fitabase.com/media/1930/fitabasedatadictionary102320.pdf).

# Data Integrity and Bias
Our dataset includes 18 CSV files with data from 30 FitBit users who agreed to share their personal tracker data via Amazon Mechanical Turk. While the dataset is reliable, original, and comprehensive, it presents several concerns:

- Sample Size and Collection Bias: The dataset is relatively small, and data collection is unevenly distributed across specific days of the week.
- Recency of Data: The data is eight years old, having been collected in 2016. This may affect its relevance to current trends and technologies.
- Lack of Critical Demographic Information: The dataset does not include demographic information such as gender. This omission is significant because Bellabeat’s products are specifically designed for women, making gender-specific insights crucial to our analysis and subsequent marketing strategies

After uploading the cleaned datasets to BigQuery, several verification steps were implemented to ensure data integrity before moving to the Analyze Phase.



```sql
SELECT *
 FROM `bellabeat-422720.bella.dailyActivity_merged` AS DailyActivity

 -- 413 records from SleepDay table
SELECT *
FROM `bellabeat-422720.bella.SleepDay_merged` 
AS SleepDay
-- 22,099 records from HourlyCalories table
SELECT *
FROM `bellabeat-422720.bella.HourlyCalories_merged` AS HourlyCalories
-- 22,099 records from HourlyIntensities table
SELECT *
FROM `bellabeat-422720.bella.HourlyIntensities_merged` AS HourlyIntensities
-- 22,099 records from HourlySteps table
SELECT *
FROM `bellabeat-422720.bella.HourlySteps_merged` AS HourlySteps
-- 1,325,580 records from MinuteMETs table
SELECT *
FROM `bellabeat-422720.bella.Nnew-minuteMETsNarrow_merged` AS MinuteMETS

-- Number of Unique Participants

-- 33 unique participants in DailyActivity table
SELECT COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.dailyActivity_merged`

-- 25 unique participants in METs table
SELECT COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.Nnew-minuteMETsNarrow_merged` AS MinuteMETS

-- 24 unique participants in SleepDay table
SELECT COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.SleepDay_merged` AS SleepDay

-- 33 unique participants in HourlyCalories, HourlyIntensities, and HourlySteps
SELECT COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.HourlyCalories_merged` AS HourlyCalories

#33
SELECT COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.HourlyIntensities_merged` AS HourlyIntensities

#33
SELECT 
COUNT(DISTINCT id) AS UserCount
FROM `bellabeat-422720.bella.HourlySteps_merged` AS HourlySteps

-- No duplicates found on DailyActivities table
SELECT Id, 
  ActivityDate, 
  TotalSteps, 
  TotalDistance, 
  TrackerDistance, 
  LoggedActivitiesDistance,	
  VeryActiveDistance,	
  ModeratelyActiveDistance,	
  LightActiveDistance,	
  SedentaryActiveDistance,	
  VeryActiveMinutes,	
  FairlyActiveMinutes,	
  LightlyActiveMinutes,	
  SedentaryMinutes,	
  Calories,
  COUNT(*) AS Dupes
FROM `bellabeat-422720.bella.NEWdailyActivity_merged` AS DailyActivity
GROUP BY Id, 
  ActivityDate, 
  TotalSteps, 
  TotalDistance, 
  TrackerDistance, 
  LoggedActivitiesDistance,	
  VeryActiveDistance,	
  ModeratelyActiveDistance,	
  LightActiveDistance,	
  SedentaryActiveDistance,	
  VeryActiveMinutes,	
  FairlyActiveMinutes,	
  LightlyActiveMinutes,	
  SedentaryMinutes,	
  Calories
HAVING Dupes > 1

###

CREATE OR REPLACE TABLE bellabeat-422720.bella.DailyActivity_cleaned AS 
SELECT
  Id,
  ActivityDate,
  TotalSteps,
  TotalDistance,
  TrackerDistance,
  LoggedActivitiesDistance,
  VeryActiveDistance,
  ModeratelyActiveDistance,
  LightActiveDistance,
  SedentaryActiveDistance,
  VeryActiveMinutes,
  FairlyActiveMinutes,
  LightlyActiveMinutes,
  SedentaryMinutes,
  Calories
FROM
  bellabeat-422720.bella.NEWdailyActivity_merged;


  ##

  -- Found 3 duplicates from SleepDay table, removed those duplicates.
SELECT
  Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed,
  COUNT(*) AS Dupes
FROM `bellabeat-422720.bella.SleepDay_merged` AS SleepDay1
GROUP BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed
HAVING Dupes = 1
ORDER BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed


  ###

-- Created user no table to clearly identify unique participant
-- Merged Daily Activity, Sleep Day, and User No tables.
-- Nulls have been set to 0.
SELECT 
  DailyActivity.Id,
  DailyActivity.ActivityDate, 
  SUM(DailyActivity.TotalSteps) AS Total_Steps, 
  SUM(DailyActivity.TotalDistance) AS Total_Distance, 
  SUM(DailyActivity.TrackerDistance) AS Total_Tracker_Distance, 
  SUM(DailyActivity.LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,	
  SUM(DailyActivity.VeryActiveDistance) AS Total_VeryActiveDistance,	
  SUM(DailyActivity.ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,	
  SUM(DailyActivity.LightActiveDistance) AS Total_LightActiveDistance,	
  SUM(DailyActivity.SedentaryActiveDistance) AS Total_SedentaryActiveDistance,	
  SUM(DailyActivity.VeryActiveMinutes) AS Total_VeryActiveMinutes,
  SUM(DailyActivity.FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
  SUM(DailyActivity.LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
  SUM(DailyActivity.SedentaryMinutes) AS Total_SedentaryMinutes,
  SUM(DailyActivity.Calories) AS Total_Calories
FROM 
  `bellabeat-422720.bella.NEWdailyActivity_merged` AS DailyActivity
GROUP BY 
  DailyActivity.Id, 
  DailyActivity.ActivityDate
####

--Hourly intensity data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT HourlyIntensities.Id, HourlyIntensities.ActivityHour, SUM(TotalIntensity) AS Total_Intensity, AVG(AverageIntensity) AS Avg_Intensity, 
    EXTRACT(YEAR FROM HourlyIntensities.ActivityHour) AS Year,
    EXTRACT(MONTH FROM HourlyIntensities.ActivityHour) AS Month,
    EXTRACT(DAY FROM HourlyIntensities.ActivityHour) AS Day,
    EXTRACT(DAYOFWEEK FROM HourlyIntensities.ActivityHour) AS DayName,
    EXTRACT(Hour FROM HourlyIntensities.ActivityHour) AS Hour,
    COUNT(*) AS Duplicates
FROM `bellabeat-422720.bella.NEWHourlyIntensities_merged` AS HourlyIntensities
GROUP BY HourlyIntensities.Id, HourlyIntensities.ActivityHour
ORDER BY HourlyIntensities.Id, HourlyIntensities.ActivityHour

###
--Hourly calories data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT Id, ActivityHour, SUM(Calories) AS Total_Calories,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
FROM `bellabeat-422720.bella.NEWHourlyCalories_merged` AS Hourly_Calories
GROUP BY Id, ActivityHour
ORDER BY Id, ActivityHour
###
--Hourly steps data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT Id, ActivityHour, SUM(StepTotal) AS Total_Steps,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
FROM `bellabeat-422720.bella.NEWHourlySteps_merged` AS HourlySteps
GROUP BY Id, ActivityHour
ORDER BY Id, ActivityHour

##
--Merged 3 hourly tables into one
SELECT HourlyIntensities.Id, HourlyIntensities.ActivityHour, SUM(TotalIntensity) AS Total_Intensity, AVG(AverageIntensity) AS Avg_Intensity, 
  SUM(HourlyCalories.Total_Calories) AS TotalCalories,
  SUM(HourlySteps.Total_Steps) AS TotalSteps,
  EXTRACT(DATE FROM HourlyIntensities.ActivityHour) AS Date,
  EXTRACT(YEAR FROM HourlyIntensities.ActivityHour) AS Year,
  EXTRACT(MONTH FROM HourlyIntensities.ActivityHour) AS Month,
  EXTRACT(DAY FROM HourlyIntensities.ActivityHour) AS Day,
  FORMAT_DATE('%A', DATE(HourlyIntensities.ActivityHour)) AS DayName,
  CAST(EXTRACT(TIME FROM HourlyIntensities.ActivityHour) AS TIME) AS Time,
  EXTRACT(HOUR FROM HourlyIntensities.ActivityHour) AS Hour
FROM `bellabeat-422720.bella.NEWHourlyIntensities_merged` AS HourlyIntensities
LEFT JOIN 
  (SELECT Id, ActivityHour, SUM(Calories) AS Total_Calories,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  FROM `bellabeat-422720.bella.NEWHourlyCalories_merged` AS Hourly_Calories
  GROUP BY Id, ActivityHour) AS HourlyCalories
ON HourlyIntensities.Id = HourlyCalories.Id
AND HourlyIntensities.ActivityHour = HourlyCalories.ActivityHour
LEFT JOIN 
  (SELECT Id, ActivityHour, SUM(StepTotal) AS Total_Steps,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAY FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
  FROM `bellabeat-422720.bella.NEWHourlySteps_merged` AS Hourly_Steps
  GROUP BY Id, ActivityHour
  ORDER BY Id, ActivityHour) AS HourlySteps
ON HourlyIntensities.Id = HourlySteps.Id
AND HourlyIntensities.ActivityHour = HourlySteps.ActivityHour
GROUP BY HourlyIntensities.Id, HourlyIntensities.ActivityHour
ORDER BY HourlyIntensities.Id, HourlyIntensities.ActivityHour

--METs per minute, extracted year, month, day, day name, hour 
-- No duplicates found 
-- Aggregated into METs per day
WITH Minute_METs AS (SELECT Id, SUM(METs) AS METs,
EXTRACT(DATE FROM ActivityMinute) AS Date,
  EXTRACT(YEAR FROM ActivityMinute) AS Year,
  EXTRACT(MONTH FROM ActivityMinute) AS Month,
  EXTRACT(DAY FROM ActivityMinute) AS Day,
  FORMAT_DATE('%A', DATE(ActivityMinute)) AS DayName,
  CAST(EXTRACT(TIME FROM ActivityMinute) AS TIME) AS Time,
  EXTRACT(HOUR FROM ActivityMinute) AS Hour,
  COUNT(*) AS Duplicates
FROM `bellabeat-422720.bella.Nnew-minuteMETsNarrow_merged` AS MinuteMETS
GROUP BY Id, ActivityMinute
ORDER BY Id, ActivityMinute)

SELECT Id, Date, SUM(METs) AS Total_METs, ROUND(AVG(METs), 2) AS Avg_METs,
  EXTRACT(YEAR FROM Date) AS Year,
  EXTRACT(MONTH FROM Date) AS Month,
  EXTRACT(DAY FROM Date) AS Day,
  FORMAT_DATE('%A', DATE(Date)) AS DayName
FROM Minute_METs
GROUP BY Id, Date;
```
# note:
DailyActivity and SleepDay tables were combined into one table.
Hourly tables (HourlyIntensities, HourlyCalories, and HourlySteps) were combined into one table.
HourlyMETSs table was aggregated into daily METs.

# Data Analysis:

```sql
###
-- Analyze Phase. Created summary stats

WITH DailyActivitySummary AS (
  SELECT 
    DA.Id,
    DA.ActivityDate, 
    SUM(DA.TotalSteps) AS Total_Steps, 
    SUM(DA.TotalDistance) AS Total_Distance, 
    SUM(DA.TrackerDistance) AS Total_Tracker_Distance, 
    SUM(DA.LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,
    SUM(DA.VeryActiveDistance) AS Total_VeryActiveDistance,
    SUM(DA.ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,
    SUM(DA.LightActiveDistance) AS Total_LightActiveDistance,
    SUM(DA.SedentaryActiveDistance) AS Total_SedentaryActiveDistance,
    SUM(DA.VeryActiveMinutes) AS Total_VeryActiveMinutes,
    SUM(DA.FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
    SUM(DA.LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
    SUM(DA.SedentaryMinutes) AS Total_SedentaryMinutes,
    SUM(DA.Calories) AS Total_Calories,
    IFNULL(SUM(SD.TotalSleepRecords), 0) AS Total_SleepRecords,
    IFNULL(SUM(SD.TotalMinutesAsleep), 0) AS Total_MinutesAsleep,
    IFNULL(SUM(SD.TotalTimeInBed), 0) AS Total_TimeInBed
  FROM 
    `bellabeat-422720.bella.2NEWdailyActivity_merged` AS DA
  LEFT JOIN 
    (SELECT
      Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed
    FROM `bellabeat-422720.bella.NEwSleepDay_merged 2`
    GROUP BY Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) AS SD
  ON DA.Id = SD.Id AND DA.ActivityDate = SD.SleepDay
  GROUP BY DA.Id, DA.ActivityDate
  ORDER BY DA.Id, DA.ActivityDate
)
SELECT 
  AVG(Total_Steps) AS AvgSteps,
  AVG(Total_Distance) AS AvgDistance,
  AVG(Total_VeryActiveMinutes) AS AvgVeryActiveMins,
  100 * SUM(Total_VeryActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS VeryActivePerc,
  AVG(Total_FairlyActiveMinutes) AS AvgFairlyActiveMins,
  100 * SUM(Total_FairlyActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS FairlyPerc,
  AVG(Total_LightlyActiveMinutes) AS AvgLightlyActiveMins,
  100 * SUM(Total_LightlyActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS LightlyActivePerc,
  AVG(Total_SedentaryMinutes) AS AvgSedentaryMins,
  100 * SUM(Total_SedentaryMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS SedentaryPerc,
  AVG(Total_Calories) AS AvgCalories,
  SUM(Total_MinutesAsleep) / SUM(CASE WHEN Total_MinutesAsleep > 0 THEN 1 ELSE NULL END) AS AvgMinsAsleep,
  SUM(Total_TimeInBed) / SUM(CASE WHEN Total_TimeInBed > 0 THEN 1 ELSE NULL END) AS AvgBedTime
FROM 
  DailyActivitySummary;

```
#  Insights based on the Analyze Phase :

According to [NBC](https://www.nbcnews.com/health/health-news/how-many-steps-day-should-you-take-study-finds-7-n1278853) , the fitness target of 10,000 steps per day is often encouraged, but a new study reveals that even 7,000 steps per day may contribute significantly to better health. The data shows moderate physical activity among most participants in our data set, with an average of 7,637.91 steps per day, which is suggesting an active lifestyle on average for the data. However, the majority of the day (about 81.33%) is spent sedentarily, indicating a potential area for health improvement by increasing more vigorous activity levels.

Participants get an average of 419.17 minutes of sleep per night, which is well within the recommended range based on [Sleep Foundation](https://www.sleepfoundation.org/women-sleep/do-women-need-more-sleep-than-men). However, the 39 minutes spent awake in bed may indicate some sleep inefficiencies, which could have an impact on overall sleep quality. Research suggests that women may require more sleep than males due to greater multitasking and brain usage, making sleep quality even more important. 


The second job of our Analyze Phase is to identify patterns and determine how strong a relationship exists between two variables. The findings returned values ranging from -1 to 1:

```sql
-- Correctly chain the CTEs
WITH CorrectedSleepDay AS (
  SELECT
    Id,
    PARSE_DATE('%Y%m %d', REGEXP_REPLACE(SleepDay, r'(\d{6})\s+(\d{2})', r'\1 \2')) AS SleepDay,
    TotalSleepRecords,
    TotalMinutesAsleep,
    TotalTimeInBed
  FROM 
    `bellabeat-422720.bella.NEwSleepDay_merged 2`
),
DailyActivity AS (
  SELECT 
    DA.Id,
    DA.ActivityDate, 
    SUM(DA.TotalSteps) AS Total_Steps, 
    SUM(DA.TotalDistance) AS Total_Distance, 
    SUM(DA.TrackerDistance) AS Total_Tracker_Distance, 
    SUM(DA.LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,
    SUM(DA.VeryActiveDistance) AS Total_VeryActiveDistance,
    SUM(DA.ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,
    SUM(DA.LightActiveDistance) AS Total_LightActiveDistance,
    SUM(DA.SedentaryActiveDistance) AS Total_SedentaryActiveDistance,
    SUM(DA.VeryActiveMinutes) AS Total_VeryActiveMinutes,
    SUM(DA.FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
    SUM(DA.LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
    SUM(DA.SedentaryMinutes) AS Total_SedentaryMinutes,
    SUM(DA.Calories) AS Total_Calories,
    IFNULL(SUM(SD.TotalSleepRecords), 0) AS Total_SleepRecords,
    IFNULL(SUM(SD.TotalMinutesAsleep), 0) AS Total_MinutesAsleep,
    IFNULL(SUM(SD.TotalTimeInBed), 0) AS Total_TimeInBed
  FROM 
    `bellabeat-422720.bella.NEWdailyActivity_merged` AS DA
  LEFT JOIN 
    CorrectedSleepDay AS SD
  ON DA.Id = SD.Id AND DA.ActivityDate = SD.SleepDay
  GROUP BY DA.Id, DA.ActivityDate
)
SELECT 
  AVG(Total_Steps) AS AvgSteps,
  AVG(Total_Distance) AS AvgDistance,
  AVG(Total_VeryActiveMinutes) AS AvgVeryActiveMins,
  100 * SUM(Total_VeryActiveMinutes) / NULLIF(SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes), 0) AS VeryActivePerc,
  AVG(Total_FairlyActiveMinutes) AS AvgFairlyActiveMins,
  100 * SUM(Total_FairlyActiveMinutes) / NULLIF(SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes), 0) AS FairlyPerc,
  AVG(Total_LightlyActiveMinutes) AS AvgLightlyActiveMins,
  100 * SUM(Total_LightlyActiveMinutes) / NULLIF(SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes), 0) AS LightlyActivePerc,
  AVG(Total_SedentaryMinutes) AS AvgSedentaryMins,
  100 * SUM(Total_SedentaryMinutes) / NULLIF(SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes), 0) AS SedentaryPerc,
  AVG(Total_Calories) AS AvgCalories,
  SUM(Total_MinutesAsleep) / SUM(CASE WHEN Total_MinutesAsleep > 0 THEN 1 ELSE NULL END) AS AvgMinsAsleep,
  SUM(Total_TimeInBed) / SUM(CASE WHEN Total_TimeInBed > 0 THEN 1 ELSE NULL END) AS AvgBedTime
FROM 
  DailyActivity;
```

Insights based on Correlation Analyses :
At 0.71, there is a high positive correlation between calories and distance.
At 0.64, there is a moderately high positive correlation between calories and distance.
At 0.62, there is a moderately high positive correlation between calories and very active minutes.
At 0.59, there is a moderately high positive correlation between calories and steps.
At 0.30, there is a low positive correlation between calories and fairly active minutes.
At 0.29, there is a negligible positive correlation between calories and lightly active minutes.
At -0.11, there is a negligible negative correlation between calories and lightly active minutes.


