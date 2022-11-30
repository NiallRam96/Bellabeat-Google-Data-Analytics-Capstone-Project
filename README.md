# Google DA Capstone Project: How Can a Wellness Technology Company Play It Smart?

Author: Niall Ramsbottom 

Date: 29th November 2022

Data source: The FitBit Fitness Tracker Data analyzed for this project can be found at https://www.kaggle.com/datasets/arashnic/fitbit. 

## Background
Founded in 2013, Bellabeat is a high-tech company that manufactures health focused smart products, with a focus on empowering women to unlock their full potential. Through collecting data on activity, sleep, stress, and reproductive health, Bellabeat’s mission is to empower women to reconnect with themselves, unleash their inner strengths and be what they are meant to be. They are now the number 1 women’s health tracker. 

## Objective
As junior analyst, my role is to analyse smart device usage data in order to gain insight into how consumers use non-Bellabeat smart devices. The findings of this analysis will then be used to guide future marketing strategies for Bellabeat. 



_Data analysis procedure:_
### 🙋 Ask
### 📋 Prepare
### 🗂️ Process
### 🛠 Analyse
### 📊 Share 
### ✔️ Act 


## 1. Ask 
**Business Task:** Analyse Fitbit data to gain insight into consumer habits and subsequently use this insight to guide marketing strategy for Bellabeat to grow. 

**Primary stakeholders:** Urška Sršen and Sando Mur, executive team members.

**Secondary stakeholders:** Bellabeat marketing analytics team.


## 2. Prepare 
**Procedure:** Data downloaded and organised using consistent naming conventions. 

**Data Source:** [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit)

**Details:** 
- The dataset contains personal fitness tracker from 30 FitBit users, all of which concsented to the submission of their data.
- Data includes minute-level output for physical activity, heart rate, and sleep monitoring. 
- The downloaded data set includes 18 individual CSV files containing daily activity, steps, and heart rate data. 

**Does the Data ROCCC?**

- **R**eliable: Data was collected using a distributed survey via Amazon Mechanical Turk and in some cases data was uploaded directly from fitness tracking device. 
- **O**riginal: The data is from 30 unique FitBit users
- **C**omprehensive: The dataset is comprehensive in regards to the many factors that it collected, i.e. physical activity, sleep, and weight data. However, the sample size is quite small and there are issues related to the amount of data within each variable. 
- **C**urrent: The data was collected between the 12th of March to the 12th of May 2016. This data is over 6 years old, thus, is not current and may not reflect the current habits of users.
- **C**ited: It is not clear if the data has been cited. 

🚩 **Limitations of dataset:** 🚩
- Small data set. Only including 30 users may limit the strength of the implications from this analysis. 
-	The format of columns and their respective cells are inconsistent. For example, data within the ‘Date’ column is not formatted consistently as some cells contain data in the AM/PM format whereas other cells contain data in the 24hr format. 
-	Although the data was allegedly collected from 30 users, initial inspections to check for unique user ID within each CSV file shows inconsistencies. Using a simple formula (=SUM(1/COUNTIF(target, range)) in Excel, the physical activity data contains 33 unique users, sleep data contains 24 unique users, and weight data contains only 8. 
-	The weight dataset contains 67 data entries, 54 of which are from 2 of the 8 unique users. Further, all of the data from one of these users was entered manually. Overall, 3 users’ data was automatically uploaded whereas the remaining 5 users’ data was entered manually. 


## 3. Process
- Files initially examined in Excel to ensure that there were no duplicates, null values, and/or missing values, and then column names were examined to ensure consistent column naming convention for legibility.

- Respective dataset (Daily Activity, Sleep Data, & Weight Data) subsequently uploaded to RStudio to be processed: 
```
Daily_activity <- read.csv("Daily_Activity_Processed.csv") 
```

- Resulting Data Frame's structure examined to ensure the CSV file uploaded successfully:
```
glimpse(Daily_activity)

head(Daily_activity)

str(Daily_activity)
```
- This shows that 'Date' appears as a chr variable, thus, it was subsequenlty converted to a date variable. Month, day, and weekday information was subtracted from this new variable:
```
library(lubridate)

Daily_activity$new_date <- dmy(Daily_activity$ActivityDate)

Daily_activity$Month <- month(Daily_activity$new_date)

Daily_activity$Day <- day(Daily_activity$new_date)

Daily_activity$Weekday <- weekdays(Daily_activity$new_date)
```
- Weekdays arranged in chronological order for clarity in visualisations. 
```
Daily_activity_updated$Weekday <- factor(Daily_activity_updated$Weekday, levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
```
- Sleep Data: Total sleep time and time in bed converted from minutes to hours (and rounded to two decimal places) for clarity in visualisations. 
```
Sleep_Data$TotalHoursAsleep <- Sleep_Data$TotalMinutesAsleep/ 60
Sleep_Data$TotalHoursAsleep <- round(Sleep_Data$TotalHoursAsleep, digits = 2)

Sleep_Data$TotalHoursInBed <- Sleep_Data$TotalTimeInBed/60
Sleep_Data$TotalHoursInBed <- round(Sleep_Data$TotalHoursInBed, digits = 2)
```

## 4. Analyse 

#### Activity Data 

Descriptive statistics for all activity variables calculated using the summary funciton: 

```
summary(Daily_activity_updated)

```
![image](https://user-images.githubusercontent.com/118458814/204821622-8d83177f-d20f-41b9-848c-f959aae39a87.png)


Next, the average steps per weekday were calculated and visualised to show any discrepancies. Mean steps per day range from 6933 to 8153, suggesting that participants are slightly more active than the average person (sugeested average daily steps is ~6886 steps (Berko et al.,2016)). Most steps are recorded on Saturdays while the least steps are recorded on Sundays, although this difference is not significant. The error bars of this plot clearly ilustrate the substantial spread of values and their deviation from the mean. This therefore hinders the conclusions that we can draw:  
```
Daily_Steps <- Daily_activity_updated %>%
  group_by(Weekday) %>%
  summarise(mean(TotalSteps), sd(TotalSteps))
  
  ggplot(Daily_Steps, aes(x = Weekday, y = Daily_Steps$`mean(TotalSteps)`, fill = Weekday)) +
  geom_bar(stat = "identity") + 
  geom_errorbar(aes(ymin=Daily_Steps$`mean(TotalSteps)`- Daily_Steps$`sd(TotalSteps)`, ymax= Daily_Steps$`mean(TotalSteps)`+ Daily_Steps$`sd(TotalSteps)`), width = .2, position = position_dodge((.9)))+
  labs(title = "Average Steps per Weekday", x = "Weekday", y = "Steps")
  ```
 ![image](https://user-images.githubusercontent.com/118458814/204844557-4298d489-20c1-451d-ab28-cf559a3cae8e.png)

In order to determine the relationship between total steps and calories burend, a correlation analysis was conducted. While there are some issues with the data, such as some participants not recording steps but still having calorie data, we can see a moderate positive association between total steps and calories burned: 

 















