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

# Insights based on Correlation Analyses :

- At 0.71, there is a high positive correlation between calories and distance.
- At 0.64, there is a moderately high positive correlation between calories and distance.
- At 0.62, there is a moderately high positive correlation between calories and very active minutes.
- At 0.59, there is a moderately high positive correlation between calories and steps.
- At 0.30, there is a low positive correlation between calories and fairly active minutes.
- At 0.29, there is a negligible positive correlation between calories and lightly active minutes.
- At -0.11, there is a negligible negative correlation between calories and lightly active minutes.


Following visualization is made using Tableau Desktop to illustrate correlations:


![177237376-3abd0cf2-0a48-41c6-b7ac-b66a361d4b00](https://github.com/asrapp/BellaBeat_project/assets/96214105/c23f96e9-a082-44b9-915a-f83363935f8e)



- Calories vs. Distance: This chart shows a clear positive correlation between calories burned and distance traveled. The trend suggests that as distance increases, so does calorie expenditure.
- Calories vs. Steps: The plot displays a similar positive trend between the number of steps taken and calories burned, indicating that more steps typically lead to higher caloric burn.
- Calories vs. METs: The relationship between METs and calories is strongly positive, highlighting that higher MET values, which indicate more intense activities, are associated with greater calorie burn.
- Calories vs. Very Active Minutes: This graph demonstrates that increased time spent in vigorous physical activity correlates with a higher number of calories burned, reinforcing the value of intense exercise for caloric expenditure


![177237694-31a79c77-9f9c-4d02-ae73-e56b216d77f4](https://github.com/asrapp/BellaBeat_project/assets/96214105/a01d6e7a-ec63-4b2d-bed1-39d1b33a9723)




- Average Calories by Hour : On average, users burn most of their calories from 5PM-7PM.
- Average Intensity by Hour : Users recorded their highest intensity levels from 5-7 PM.
- Average Steps by Hour : Users walk the most by 5PM-7PM.


![177237874-04df95e2-d879-42c5-a75c-20efe87985aa](https://github.com/asrapp/BellaBeat_project/assets/96214105/276b2b34-f573-4a77-8120-b4a4f70b65b2)



- Calories Burned: Users burn the most calories on Tuesdays and the fewest on Thursdays.
- METs: Tuesdays see the highest METs, while Thursdays have the lowest.
- Very Active Minutes: Users are most active on Tuesdays and least active on Thursdays.
- Steps Taken: The highest number of steps is recorded on Tuesdays, and the fewest on Fridays.

# Recommendations
- Update the Dataset: The current dataset is eight years old. To enhance relevance and accuracy, Bellabeat should consider updating the dataset with more participants and diverse demographic data, including gender, age, location, and physiological metrics like height and weight. This will enable a broader and more inclusive analysis.
- Integrate Comprehensive Health Metrics: The addition of heart rate and weight logs could significantly enrich the dataset. Expanding the integration of these metrics within Bellabeat’s products can help users track their health more comprehensively. Enhancing the app to sync seamlessly with devices that monitor these metrics will provide users with a holistic view of their health.
  
- Leverage Insights for Marketing and Product Enhancement:
- Caloric Burn Prediction: Utilize the positive correlation between calories burned and factors like steps, distance, METs, and very active minutes to develop --predictive analytics features. This tool could forecast caloric burn based on user activity patterns, encouraging users to stay active.
- Targeted Notifications: Implement notifications timed around peak activity periods, particularly from 5-7 PM, to encourage users to engage in moderate to intense physical activities. This aligns with the data showing when users are most active.
  
- Optimize Sleep Health Features:
- Sleep Optimization Notifications: Since the data suggests an average 40-minute period to fall asleep, integrating sleep optimization tips and reminders could help users enhance sleep quality.
- Custom Sleep Goals: Allow users to set and adjust sleep duration goals based on personal data and trends observed in the app.
  
- Sedentary Lifestyle Interventions:
- Activity Reminders: For users frequently in a sedentary state, automatic reminders to engage in light or moderate physical activity after prolonged periods of inactivity could be beneficial.
- Activity-Based Notifications: Since peak activity times are in the late afternoon and early evening, timed reminders or motivational messages could help maintain or increase user activity levels during these times.
  
- Incentivize Fitness Goals:
- Rewards System: Develop a rewards system that acknowledges user milestones with virtual badges or points that can be shared on social media. This could enhance user engagement and motivation.
- Social Challenges: Implement features that allow users to participate in fitness challenges with friends or the Bellabeat community, fostering a supportive and competitive environment.
