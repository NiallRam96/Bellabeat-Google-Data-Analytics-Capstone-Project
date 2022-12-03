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

In order to determine the relationship between total steps and calories burend, a correlation analysis was conducted. While there are some issues with the data, such as some participants not recording steps but still having calorie data, we can see a moderate to strong positive association (r = 0.591) between total steps and calories burned: 
```
Daily_activity_updated%>%
  group_by(Weekday) %>%
  summarize(mean(TotalSteps), sd(TotalSteps), mean(Calories), sd(Calories), cor(TotalSteps,Calories))
  ```
  
  ![image](https://user-images.githubusercontent.com/118458814/204848114-2ae2b4cd-180a-4c19-9663-81188bd09977.png)
  
  For further insight into how participants allocated their time, total activity times were converted to percentages and visualised in a pie chart. As we can see, participants spent 81% of their time sedentary. However, further inspection of the dataset shows that 79 of 940 (8.4%) data entries were 100% sedentary (i.e. 1440 minutes sedentary) which may have skewed data. Nonetheless, this gives an interesting insight as to how users spend their time:  
  
![image](https://user-images.githubusercontent.com/118458814/204849986-a4af8df3-9e96-41c2-9678-7d16fbb9f609.png)

Building on this, data was analysed to determine which days participants were more likely to be sedentary. Results suggest that Monday has the highest average time spent sedentary, although there is no significant difference between any of the days (see error bars):
![image](https://user-images.githubusercontent.com/118458814/204856089-698ce705-d908-4be2-8604-115b3064b8ed.png)

However, the analysis also shows that participants are more likely to be very active on Monday, Tuesday, and Satruday, with slight dips mid week and on Sundays. Although these averages are not significantly different, it provides an insight into the structure of the participants' routine. Several unknwown variables, such as the participants' occupations etc, make it difficult to make more detailed implications. 

![Average Time Spent Very Active Per Day](https://user-images.githubusercontent.com/118458814/205461594-bddf2f76-6065-4004-83c5-70d6b1570c93.png)

#### Sleep Data
As mentioned above, sleep data was only gathered from 24 participants and tracked their total time in bed and total time spent asleep. Data was first summarised to get initial insights. 

This data illustrates that participants averaged 6.99 hours of sleep per night but their average time spent in bed was 7.64 hours, suggesting that particpants spent an average of ~40 minutes awake while in bed. This time awake may be spent prior to falling asleep and/or may be due to the participant waking up during the night, although we can't be certain as we do not have data to confirm this. However, the summary stats highlight key issues in the data set, such as large ranges in total time in bed and total sleep time. 

![image](https://user-images.githubusercontent.com/118458814/205462163-f0e67b65-87be-4b00-929f-ff99b670f4a6.png)


To dive deeper, I analysed and visualised the average sleep time per weekday. Results show that the participants only averaged the recommended 7-8 hours (Mayo Clinic, National Sleep Foundation) of sleep per night on two occasions, Wednesday and Sunday. Getting more sleep on Sunday makes sense as most people are preparing for a week of school, university, or work. Similarly, these participants may have experienced mid-week fatigue from their respective schedule. 

![Average Sleep Time Per Week](https://user-images.githubusercontent.com/118458814/205462547-8698b37e-863a-44c0-923d-c90d0d8822e5.png)

In accordance with the above analysis, I examined the average time spent in bed per weekday. As you can see, this graph follows an identical pattern to the above. 

![Average Time Spent in Bed](https://user-images.githubusercontent.com/118458814/205463008-7a22624b-4a68-4c95-9263-971c1bcb5ff9.png)

This data provides an interesting insight into the sleep patterns of participants and highlights additonal areas to be examined that may provide more context to these trends, as discussed below in section 6. 

#### Weight Data
Finally, the weight data from 8 unique participants was analysed. Collected data included weight (Kg & pounds), BMI, fat %, and the collection method (automatic or manual). Initial descriptive statistics highlight some important issues. For instance, over 60% of data was logged manually and the means of partipants may be impacted by an outlier (highlighted below). 

![image](https://user-images.githubusercontent.com/118458814/205463230-d0047d20-1a21-459f-962c-85f4ae694be4.png)

![image](https://user-images.githubusercontent.com/118458814/205463581-f97aeb14-1381-4524-837e-c063dabe8076.png)








