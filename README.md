# Google Data Analytics Certificate Capstone Project
### Bike Data Project




# Background
This case study serves as the final project for the 9-part [Google Data Analytics Professional Certificate course](https://www.coursera.org/professional-certificates/google-data-analytics). The Python portion of the analysis works off of sample code, which I adapted for the four datasets I used and added additional analysis to.


In this scenario, I am a junior data analyst working for a bike-share company. The company has two customer types: casual users and annual members. I am tasked with answering the first question of a series of questions designed to form a marketing strategy to encourage casual users to become full-time members: How do annual members and casual riders use bikes differently?


# Tools Used
- Excel: Used for initial spotcheck/analysis of the data sources.
- Google Collab/Python: This was the backbone of my analysis, allowing me to combine the large datasets to produce relevant insights.
- Tableau: This was used to turn my insights into easy-to-understand graphs and charts detailing my findings.


# The Analysis
In the Google Analytics program, the phases of data analysis are broken down into distinct phases:
1. Ask
2. Prepare
3. Process
4. Analyze
5. Share
6. Act


## Ask Phase
In this phase, the goal is to key in on the main problem we are trying to solve and the business decisions this analysis will affect. In this case, while the task we have been given is just to analyze the difference between casual and annual members, we have to make sure that our final deliverable allows the key stakeholders within the firm to answer the next questions:
1. Why would casual riders buy the full-time membership?
2. How can the firm use digital media to influence casual riders to become members?




## Prepare Phase
In this phase, we make sure that the data we are using is well-sourced, credible, unbiased, and relevant to the question at hand.


The data used in this project comes from [this provided collection](https://divvy-tripdata.s3.amazonaws.com/index.html) of anonymized bike trip data used under a [data license agreement](https://divvybikes.com/data-license-agreement) between the City of Chicago and Divvy/Lyft.


After confirming that the data was from a trusted source, I wanted to open the four chosen datasets to make sure it had the information necessary to answer the case question. All four datasets had information on the membership status of the person who took the trip, trip time data, and station to/from data - all of which would be useful in differentiating between the two groups.


## Process
In this phase, we analyze the datasets we are using to see if any cleaning/manipulations need to be made to the data to ensure its integrity.


After opening all four files in Excel, it was clear that the first step would be to rename the columns in the 2019 Q2 and 2020 Q1 files to match those in the 2019 Q3 and 2019 Q4 files. The image below shows the name changes:
![Image of column name changes]("column_changes.png")


After making the column name changes, I uploaded the four files to Google Colab using the pandas library:
```python
import pandas as pd
q2_2019 = pd.read_csv("/content/Divvy_Trips_2019_Q2.csv")
q3_2019 = pd.read_csv("/content/Divvy_Trips_2019_Q3.csv")
q4_2019 = pd.read_csv("/content/Divvy_Trips_2019_Q4.csv")
q1_2020 = pd.read_csv("/content/Divvy_Trips_2020_Q1.csv")
```
The first important cleaning step I took was to drop every column from the 2020 Q1 file not in any of the 2019 data:
```python
q1_2020_test = q1_2020.drop(columns=['start_lat', 'start_lng', 'end_lat', 'end_lng', 'birthyear', 'gender',
'tripduration'], errors='ignore')
```


After combining the four datasets:
```python
all_2019_trips = pd.concat([q2_2019, q3_2019,q4_2019], ignore_index=True)


all_trips = pd.concat([all_2019_trips,q1_2020_test], ignore_index=True)
```
An analysis of the usertype column shows that the combined dataset refers to members and casual users in two different ways:
```python
all_trips['usertype'].value_counts()
```
![Output of above code]("C:\users\guske\Pictures\Screenshots\4_member_names.png")


In order to fix this I set Subscriber to member and Customer to casual using this code:
```python
all_trips['usertype'] = all_trips['usertype'].replace({
'Subscriber': 'member',
'Customer': 'casual'
})
```


Lastly, as part of the next phase, a column was created to use the trip start and end times to calculate a trip duration. Sorting this column reveals that some of these values are negative, meaning they must be removed from the dataset:
```python
#Using Boolean Masking to see how many negative values there are
all_trips['ride_length'].sort_values(ignore_index=True)


neg_ride = all_trips['ride_length'] < 0


len(all_trips[neg_ride])


# Code to make a copy of the dataset with any rows with negative trip duration values excluded
all_trips_v2 = all_trips[(all_trips['from_station_name'] != "HQ QR") & (all_trips['ride_length'] >=
0)].copy()
```


## Analyze Phase
In this phase, now that we have clean data and a clear business objective in mind, we can then perform an analysis based on the data.


Since we are trying to glean the difference between casual users and members, we need our analysis should focus on comparing usertype to the other relevant fields in our combined dataset:
1. Time of Trip
2. Trip Duration
3. To/From Station


For #1 we will use the start_time column to break out the datetime figures into different related columns:
```python
#Code to use start/end time to make date, month, day, year, day of the week columns
all_trips['date'] = all_trips['start_time'].dt.date
all_trips['month'] = all_trips['start_time'].dt.month
all_trips['day'] = all_trips['start_time'].dt.day
all_trips['year'] = all_trips['start_time'].dt.year
all_trips['day_of_week'] = all_trips['start_time'].dt.day_name()
```


For #2, we use the start_time and end_time columns to make a new trip duration column for the entire dataset:
```python
#Code to make ride_length column
all_trips['ride_length'] = (all_trips['end_time']-all_trips['start_time']).dt.total_seconds()
```


Now that we have all the columns needed for analysis, we'll need to use the .groupby and .agg methods to look at this data:


```python
# Compare members and casual users
print("\nRide length statistics grouped by usertype:\n",
all_trips_v2.groupby('usertype')['ride_length'].agg(['mean', 'median', 'max', 'min']))
```
![Summary statistics grouped by usertype]("C:\Users\guske\Pictures\Screenshots\member_mean.png")


As this shows, casual ride times are much higher per trip than those of members. We can then see how the day of the week impacts average ride length:


```python
# The days of the week need to be ordered for correct sorting for analysis
days_order = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]
all_trips_v2['day_of_week'] = pd.Categorical(all_trips_v2['day_of_week'],
categories=days_order, ordered=True)


# Now, run the aggregation again to see the sorted result
print("\nAverage ride length (sorted by day of week):\n", all_trips_v2.groupby(['usertype',
'day_of_week'])['ride_length'].mean())
```
![Average Ride Length]("C:\users\guske\Pictures\Screenshots\avg_ride_len.png")


I performed more analysis in this manner and then used .to_csv to download this grouped data for use in the next phase of analysis, share:
```python
# Saving user/day of week by ride len for use in Tableau
counts = all_trips_v2.groupby(['usertype',
'day_of_week'])['ride_length'].mean().reset_index()


counts.to_csv('avg_ride_length.csv', index=False)
```
## Share Phase
In this penultimate phase, the goal is to take our analysis and lay it out in a way that answers our initial question, goes through key aspects of the analysis, and does it in a clear and compelling way.


I will use graphs and visualizations I made using the tool Tableau and csv files I exported from Python (see previous section's code).


There are two different measures: number of rides and ride duration. There are three different ways to group these: month, day of the week, and location/station name.


An analysis by month shows that there isn't a discernible difference in trend by group, beyond the fact that there are many more rides taken by members:
![Members vs Casuals by number of rides by month]("C:\Users\guske\Pictures\Screenshots\Tableau_Chi\Rides_month.png")


However, an analysis of weekdays shows that casual users ride for much longer, and are more likely to ride on the weekend vs the members:
![Weekday data]("C:\Users\guske\Pictures\Screenshots\Tableau_Chi\Rides_both.png")


Given the analysis performed in these first two parts, it's clear that there are two key differences:
1. Casual users are more likely to ride on the weekends than on weekdays, while members ride more on weekdays than on weekends.


2. Casual users have much longer ride durations than members.


Putting these two pieces of information together, it seems like members are more likely to use this as a daily transportation service to get to and from work/school. Meanwhile, longer weekend trips for casual users would point towards days out in the city and potential use by tourists.


In order to test this theory, in my final visualization, I used the longitude/latitude data from the 2020 Q1 data to map the 5 most common station destinations for both casual users and members:
![Chicago Map]("C:\Users\guske\Pictures\Screenshots\Tableau_Chi\Map.png")


In the end, this hypothesis is confirmed by the map. The orange member stations are all centered around a key transit hub - Chicago Union Station, and a commercial district further away from the lake. Meanwhile, the top-five casual user stations are by popular tourist/recreational spots closer to the lakeshore, like the Navy Pier and the Chicago History Museum.








## Act Phase
In this final phase, we take our findings and present our final conclusions. In this case, since this analysis was the first part of a multi-part effort to craft a marketing strategy, we can focus our final conclusions/recommendations on preparing our co-workers to answer the next two questions:
1. Why would casual riders buy Cyclistic annual memberships?


2. How can Cyclistic use digital media to influence casual riders to become members?


As our analysis showed, non-members ride for much longer per trip, riding for about an hour per trip. Looking at the [Divvy website today](https://divvybikes.com/pricing), a ride of about an hour would cost about $9.55 more for non-members than for members. This difference jumps up to $19.8 when using an E-bike.


Given the cost of a membership for new members is only $99/year, it only takes about ~10 similar rides per year for the membership to be worth it. Therefore, one recommendation would be to look further into how many times a year the same non-member uses the service. This could most likely be done by matching the cardholder's name in the firm's internal data.


Our analysis also showed the difference in the number of rides by weekday. Casual users ride the most on weekends, while members ride more often on weekdays. Currently, our dataset has incomplete birth year data (some for the 2019 data and none for the 2020 Q1 data). We could potentially see if we have more internal data on age, or if we could use the user's names to match to third-party data.


Given this weekday disparity, non-members may be younger people who don't need to commute to work every day but are interested in getting to weekend leisure locations. This could potentially lead to a strategy of digitally targeting area-college students.


Lastly, we saw that casual users tended to bike to stations in more recreational/touristy areas when compared to the member stations in more commercial/transit-heavy areas. Like the weekday discrepancy, this points to a difference of purpose, where casual members use bikes more for leisure activities. Therefore, further analysis could be performed on whether a marketing partnership, or advertising on the websites of these businesses in these key locations (ex. the Chicago History Museum) would drive membership sign-ups.


# What I Learned
From working on this project, I've learned many new ways to use Python, specifically with the Pandas library, and how to tell a data story using Tableau:


🧩 Working With Large Databases: Each of the four datasets ran incredibly slowly when I attempted to filter/edit in Excel. This project only worked because I used a tool like Python with Pandas or SQL.


📊 Data Aggregation: Used .groupby and .agg methods to find key insights in the data.


💡 Analytical Wizardry: Leveled up my real-world puzzle-solving skills, turning questions into actionable, insightful SQL queries.


## Closing Thoughts
I had a blast getting to really dive deep into Python for the first time. I've had experience doing similar work with SQL, and it was fun seeing things I know how to do with one tool done in a different way in another. While I still think that for a task like this, SQL is a bit easier, I also saw how much more powerful Python can be, given the myriad available libraries.


For my next project, I think I'll try to incorporate analysis using spreadsheets, SQL, and Python!











