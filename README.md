# Cyclistic-Google-Data-Analytics-Capstone
Analysis on historic Cyclistic Bike Share ride data in order to understand the behaviour of casual riders and annual members, with the ultimate goal of designing a new marketing strategy to convert casual riders into annual members.
<br><br><br>

## Scenario
In this case study for the Google Data Analytics Course Capstone, my role is a junior analyst within the marketing analytics team for a bike-share company Cyclistic in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Consequently, we want to understand how casual riders and annual members use Cyclistic bikes differently. These insights will support a new marketing strategy to convert casual users into annual members. 
<br><br>
Cyclistic is a bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can’t use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each day.
<br><br><br>

## Ask
The business task is to analyze historical bike trip data to understand the behaviour of casual riders and annual members, with the ultimate goal of designing a new marketing strategy to convert casual riders into annual members. This task is essential for Cyclistic’s growth and success, as it focuses on maximizing annual memberships, which have been identified as more profitable for the company.
<br>
Business task: Three questions will guide the future marketing campaign:
<br>
  1. How do annual members and casual riders use Cyclistic bikes differently?
  2. Why would casual riders buy Cyclistic annual memberships?
  3. How can Cyclistic use digital media to influence casual riders to become members?

Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.
<br><br><br>

## Prepare 
For this case study, I used 12 months of Cyclistic’s 2025 bike ride data (01/01/2025 to 31/01/2025) from the following Divvy database: https://divvy-tripdata.s3.amazonaws.com/index.html.
<br><br>

IS THE DATA ROCCC?
<br>
✅ Reliable - YES, after removing some duplicates and invalid rides, the data is accurate and unbiased. It is also first party data, increasing the reliability. <br>
✅ Original - YES - the data was downloaded from the original public data source by Motivate LLC.<br>
✅ Comprehensive - YES - despite some missing data, I feel the dataset is extensive and provides more than enough information for effective analysis. Other useful information could include postcodes, gender and ages of members and casual riders, however this would cause privacy concerns.<br>
✅ Current - YES - the data is current and updated on a month basis.<br>
❌ Cited - NO 
<br><br>

**Tools Used**
- SQL
- Tableau
<br><br><br>

## Process
I used MySQL Workbench 8.0 as I had this on my desktop. Here I was able to successfully import the 12 CSV files into MySQL Workbench 8.0 and named them according to the appropriate months (e.g. “01_2025” for January 2025 data, “02_2025” for February 2025 data etc).
<br><br>
I then created a master table called all_trips_2025 that I could later insert the data from the imported monthly tables.
<br><br>
Now I have my master all_trips_2025 table, I inserted all data from the 12 imported monthly tables:
```sql
INSERT INTO `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	SELECT * FROM `Cyclistic Capstone 2025 Data`.`01_2025`;

INSERT INTO `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	SELECT * FROM `Cyclistic Capstone 2025 Data`.`02_2025`;
    
INSERT INTO `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	SELECT * FROM `Cyclistic Capstone 2025 Data`.`03_2025`;
```
...and so on for all 12 tables. 

<br><br>
I then viewed the data information to check for any issues: 
<br>
<img width="321" height="229" alt="Screenshot 2026-06-02 at 11 30 13" src="https://github.com/user-attachments/assets/42e98681-7310-4ecc-b932-6333ca1dbd22" />
<br>
This shows the started_at and ended_at columns as text which is incorrect.
It also showed ride_id as text, however VARCHAR would be more appropriate. 
<br><br>
Consequently, I modified the started_at and ended_at columns from text to DATETIME(3). I used DATETIME(3) as the values include milliseconds:

```sql
ALTER TABLE `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	MODIFY started_at DATETIME(3),
	MODIFY ended_at DATETIME(3);
  ```
<br>
Before modifying ride_id I counted the max length of ride IDs which showed 16. Therefore I changed the ride_id column from text to VARCHAR:

```sql
--counting ride_id length:
SELECT MAX(CHAR_LENGTH(ride_id)) AS max_length
FROM all_trips_2025;
```

```sql
--changing the ride_id column from text to VARCHAR:
ALTER TABLE all_trips_2025
MODIFY ride_id VARCHAR(16);
```

It was also revealed that there were no keys/indexes within the table. Due to this and a large dataset, I consistently had Error 1114 when trying to run queries.  Consequently, I created an index for the ride_id column to make queries more efficient by helping to find, sort and group data quicker: 
```sql
CREATE INDEX idx_ride_id
ON all_trips_2025 (ride_id);
```
<br><br>

**Removing Duplicates**
Checking for duplicate values:
```sql
SELECT ride_id,
	COUNT(*) AS occurances
FROM `Cyclistic Capstone 2025 Data`.`all_trips_2025`
    GROUP BY ride_id
		HAVING COUNT(*) > 1;
-- used HAVING instead of WHERE as HAVING filters groups after grouping, WHERE filters before
```
This showed that there are 36,690 records, meaning there are 18,345 duplicate records based on ride_id. Each duplicated ride_id occurred exactly twice. 
<br>
Before excluding duplicate records, I viewed a sample of the duplicates to double check they were exact copies:
```sql
SELECT COUNT(*) AS conflicting_ride_ids
FROM (
   SELECT ride_id
   FROM all_trips_2025
   GROUP BY ride_id
   HAVING COUNT(DISTINCT member_casual) > 1) AS temp;
```

This showed that some records contained conflicting membership classifications (the same ride ID appearing as both member and casual). Therefore, I counted how many ride IDs appeared as both member and casual: 
```sql
SELECT COUNT(*) AS conflicting_ride_ids
FROM (SELECT ride_id
		FROM all_trips_2025
		GROUP BY ride_id
		HAVING COUNT(DISTINCT member_casual) > 1 ) AS temp;
```

This showed that there were 136 ride IDs with conflicting membership classifications. As this is only less than 0.01% of the data and membership classification is a key variable in the analysis, the identified conflicting ride IDs were excluded to prevent misclassification bias.
<br><br>

**Checking Null Values**<br>
```sql
SELECT 
COUNT(*) - COUNT(ride_id) AS ride_id_nulls,
COUNT(*) - COUNT(rideable_type) AS rideable_type_nulls,
COUNT(*) - COUNT(started_at) AS started_at_nulls,
COUNT(*) - COUNT(ended_at) AS ended_at_nulls,
COUNT(*) - COUNT(start_station_name) AS start_station_name_nulls,
COUNT(*) - COUNT(start_station_id) AS start_station_id_nulls,
COUNT(*) - COUNT(end_station_name) AS end_station_name_nulls,
COUNT(*) - COUNT(end_station_id) AS end_station_id_nulls,
COUNT(*) - COUNT(start_lat) AS start_lat_nulls,
COUNT(*) - COUNT(start_lng) AS start_lng_nulls,
COUNT(*) - COUNT(end_lat) AS end_lat_nulls,
COUNT(*) - COUNT(end_lng) AS end_lng_nulls,
COUNT(*) - COUNT(member_casual) AS member_casual_nulls
FROM `Cyclistic Capstone 2025 Data`.`all_trips_2025`;
```

Initial NULL-value checks returned 0 missing values. Further investigation showed that missing station information had been imported as empty strings rather than SQL NULLs:
```sql
SELECT
SUM(ride_id = '') AS ride_id_empty,
SUM(rideable_type = '') AS rideable_type_empty,
SUM(started_at = '') AS started_at_empty,
SUM(ended_at = '') AS ended_at_empty,
SUM(start_station_name = '') AS start_station_name_empty,
SUM(start_station_id = '') AS start_station_id_empty,
SUM(end_station_name = '') AS end_station_name_empty,
SUM(end_station_id = '') AS end_station_id_empty,
SUM(start_lat = '') AS start_lat_empty,
SUM(start_lng = '') AS start_lng_empty,
SUM(end_lat = '') AS end_lat_empty,
SUM(end_lng = '') AS end_lng_empty,
SUM(member_casual = '') AS member_casual_empty
FROM all_trips_2025;
```
This showed the following columns had missing values:
<br>
start_station_name - 943,271 <br>
start_station_id - 943,207 <br>
end_station_name - 1,019,816 <br>
end_station_id - 1,019,573 <br>
<br>
This shows that the null/missing values occurred in the starting and ending station names and IDs. 
For station analysis, I will use start_station_name and end_station_name rather than ids.
<br>
I will see if any starting and ending station names have null/missing values:
```sql
--start stations:
SELECT COUNT(*) AS missing_start_station_rows
FROM all_trips_2025
WHERE (start_station_name IS NULL OR start_station_name = '');

--end stations:
SELECT COUNT(*) AS missing_end_station_rows
FROM all_trips_2025
WHERE (end_station_name IS NULL OR end_station_name = '');
```
This showed that:
There are 943,271 rows that had null/missing values for start_station_name.
There are 1,019,816 rows that had null/missing values for end_station_name.
<br>
Rides that had null/missing values in station name columns will be excluded from any relevant analysis.
<br><br>

**Checking for Invalid Rides**<br>
Invalid rides were identified as records where ended_at <= started_at. Counts were calculated using distinct ride_id values to avoid inflating results due to duplicate records:
```sql
SELECT COUNT(DISTINCT ride_id) AS invalid_rides
FROM all_trips_2025
WHERE ended_at <= started_at;
```
There are 10,751 invalid rides. These records were excluded from subsequent analyses to ensure ride-duration analysis reflected valid trips only.
<br><br>

**Adding hour_start, day_start and month_start columns**<br>
I extracted useful information from the started_at column, such as the hour each trip began, day of the week and month, and ride length:
```sql
ALTER TABLE `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	ADD COLUMN start_hour TINYINT UNSIGNED
		GENERATED ALWAYS AS (EXTRACT(HOUR FROM started_at)) STORED,
	ADD COLUMN start_weekday TINYINT UNSIGNED
		GENERATED ALWAYS AS (WEEKDAY(started_at) + 1) STORED,
    ADD COLUMN start_month TINYINT UNSIGNED
		GENERATED ALWAYS AS (EXTRACT(MONTH FROM started_at)) STORED,
	ADD COLUMN ride_length int
		GENERATED ALWAYS AS (TIMESTAMPDIFF(MINUTE, started_at, ended_at)) STORED;
```
<br>
For ease of future dashboard use, I created two extra columns to display the week and month name (e.g. Monday instead of 1 etc):

```sql
ALTER TABLE `Cyclistic Capstone 2025 Data`.`all_trips_2025`
	ADD COLUMN weekday_name VARCHAR(10)
		GENERATED ALWAYS AS (DAYNAME(started_at)) STORED,
    ADD COLUMN month_name VARCHAR(10)
		GENERATED ALWAYS AS (MONTHNAME(started_at)) STORED;
```
<br><br>

**Checking member/ride type data**<br>
Now, I will check how many ride types and member types there are to ensure the data is valid:

```sql
--member types:
SELECT COUNT(member_casual)
FROM all_trips_2025
GROUP BY member_casual;

--ride types:
SELECT rideable_type
FROM all_trips_2025
GROUP BY rideable_type;
```

The results showed only two member types: member or casual, which is correct.
<br>
The results also showed two ride types: classic_bike or electric_bike. Older datasets often included a third category ‘docked_bike’, so I will check if there is any missing data:

```sql
SELECT COUNT(*) AS missing_rideable_type
FROM all_trips_2025
WHERE rideable_type IS NULL
   OR rideable_type = '';
```

The results showed 0 null/missing values, so I can be confident that this data is valid.
<br><br>

**Checking Ride Duration Data**<br>
I will check if ride times are less than a minute or longer than a day, excluding earlier identified invalid rides:
```sql
SELECT COUNT(DISTINCT ride_id) AS rides_over_24_hours
FROM all_trips_2025
WHERE ended_at > started_at
  AND ride_length_mins >= 1440;

-- viewing sample data:
SELECT DISTINCT ride_id,
       rideable_type,
       member_casual,
       started_at,
       ended_at,
       ride_length_mins
FROM all_trips_2025
WHERE ended_at > started_at
  AND ride_length_mins >= 1440
LIMIT 1000;
```
The results showed there were 10,813 rides over 24 hours long. The sample showed some rides lasting 24 hours and others 1000+, suggesting it is likely there are both valid long rides (24 hour rides) and invalid rides (such 1000+ hour rides) within the data. Therefore, I will exclude rides over 24 hours from duration-based analysis.
<br><br>
I will also check rides shorter than 1 minute and view sample data:
```sql
SELECT COUNT(DISTINCT ride_id) AS short_rides
FROM all_trips_2025
WHERE ended_at > started_at
  AND ride_length_mins <= 1;

--viewing sample data:
SELECT DISTINCT ride_id,
       rideable_type,
       member_casual,
       started_at,
       ended_at,
       TIMESTAMPDIFF(MINUTE, started_at, ended_at) AS ride_length_mins
FROM all_trips_2025
WHERE ended_at > started_at
  AND ride_length_mins <= 1
LIMIT 1000;
```
The results showed there are 211,052 rides lasting 1 minute or less. Rides under 2 minutes were excluded from duration-based analysis because they may represent bike checks, immediate returns, accidental unlocks or system errors.
<br><br>

**Creating a new table with cleaned data**<br>
Now I will create a new table with clean data that excludes: duplicate rows, ride IDs with conflicting membership classifications, invalid rides and rides less than 2 minutes and over 24 hours:
```sql
CREATE TABLE trips_2025_clean AS
SELECT DISTINCT *
FROM all_trips_2025
WHERE ended_at > started_at
  AND ride_length_mins > 1
  AND ride_length_mins < 1439
  AND ride_id NOT IN (SELECT ride_id
			  FROM all_trips_2025
			  GROUP BY ride_id
			  HAVING COUNT(DISTINCT member_casual) > 1);
```

Now I have the clean table, I exported the data into a CSV and uploaded it to Tableau.

There are now 4,411,435 total rides included within the analysis.
<br><br><br>


## Analyse & Share

#### Tableau Dashboard
<br><br>
View the dashboard on Tableau Public:

[Cyclistic Bike Share Analysis](https://public.tableau.com/app/profile/rebecca.jeffery2022/viz/Cyclistic-GoogleDataAnalyticsCapstone_17812633365350/Story1)
<br><br><br>


#### Number of Members vs Casual Riders
```sql
SELECT member_casual,
       COUNT(DISTINCT ride_id) AS total_rides
FROM trips_2025_clean
GROUP BY member_casual;
```
<img width="200" height="300" alt="Member class" src="https://github.com/user-attachments/assets/ad74ac2d-a289-4095-9f43-9777bf30d237" />

1,586,729 (36%) casual users in total.
2,824,706 (64%) members in total. 
This shows that the majority of Cyclistic users are members. 
<br><br><br>


#### Total Average Ride Length: Members vs Casual
```sql
SELECT member_casual,
       AVG(ride_length_mins) AS avg_ride_length_mins,
       MIN(ride_length_mins) AS min_ride_length_mins,
       MAX(ride_length_mins) AS max_ride_length_mins
FROM (SELECT DISTINCT ride_id,
           member_casual,
           ride_length_mins
     FROM trips_2025_clean) AS trips
GROUP BY member_casual;
```
<img width="1716" height="302" alt="Avg Ride Lngth - Member Class" src="https://github.com/user-attachments/assets/855d5898-7164-4280-b7cc-4684b2c825bb" />
This shows that on average casual riders had the longest rides (19.8 minutes) compared to members (12 minutes). 
<br><br><br>

#### Type of bike used: Members vs Casual
```sql
SELECT rideable_type,
      member_casual,
      COUNT(DISTINCT ride_id) AS rides
FROM trips_2025_clean
GROUP BY rideable_type, member_casual;
```

<img width="400" height="400" alt="Type of Bike" src="https://github.com/user-attachments/assets/866cc262-ba58-4aa5-bd17-bcf3de45cf00" />

This shows that both members and casual users prefer electric bikes. 
However, it would be useful to know Cyclistic’s stock of how many electric bikes in comparison to classic bikes they have available for users to use. 
<br><br><br>

#### Average Ride Length by Bike Type
```sql
SELECT rideable_type,
       AVG(ride_length_mins) AS avg_ride_length_mins
FROM trips_2025_clean
GROUP BY rideable_type;
```
<img width="1716" height="302" alt="Avg Ride Length - Bikes" src="https://github.com/user-attachments/assets/ebb38411-8d9a-468a-a4a5-73eb7791d576" />

This shows that overall rides using classic bikes had a higher average ride length in comparison to electric bikes. This is interesting as the previous table ‘Type of Bike Used: Members vs Casual Riders’ shows that both members and casual riders used electric bikes more. Consequently, it is possible that riders (both casual riders and members) who use classic bikes do so for exercise and fitness.
<br><br><br>


#### Peak Times (start hour of rides) and Average Ride Lengths: Members vs Casual Riders
<br>

```sql
SELECT start_hour,
       member_casual,
       COUNT(DISTINCT ride_id) AS rides,
       ROUND(AVG(ride_length_mins), 2) AS avg_ride_length_mins
FROM trips_2025_clean
GROUP BY start_hour, member_casual
ORDER BY start_hour, member_casual;
```
<br>
<img width="500" height="400" alt="Hourly stats" src="https://github.com/user-attachments/assets/f2dad6e6-1b82-4900-8aaa-38022735e3e2" />


The results showed the peak hours to be:<br>
*Members* - 4pm, 5pm, 6pm<br>
*Casual users* - 4pm, 5pm, 6pm
<br><br>
This demonstrates that both members and casual riders use Cyclistic mostly in the late afternoon/early evening, likely for work or school commutes. 
<br><br>
The chart also shows that for casual riders, there was a peak average length of ride between 10:00 and 14:00.
<br><br><br>


#### Peak Days of Use and Average Ride Lengths: Members vs Casual Riders

```sql
SELECT weekday_name,
       member_casual,
       COUNT(DISTINCT ride_id) AS rides,
       ROUND(AVG(ride_length_mins), 2) AS avg_ride_length_mins
FROM trips_2025_clean
GROUP BY weekday_name, member_casual
ORDER BY weekday_name, member_casual;
```

<img width="500" height="400" alt="Weekday stats" src="https://github.com/user-attachments/assets/6f9fa9c6-e4a4-4362-b86a-953844d1888d" />


<br>
The results showed the peak days:<br> 

*Members* - Tuesday, Wednesday and Thursday - since covid leading to the increase of hybrid working and more people working from home, Tuesday Wednesday and Thursday these are popular days to go in the office and Monday and Friday work from home. This again suggests that members use the service mostly for commuting to and from work. <br>
*Casual users* - Friday, Saturday and Sunday - this suggests casual users use the Cyclistic service mostly over the weekends.
<br><br>
The chart also shows that although ride of length remained more consistent for members throughout the week, the average length of rides peaked on Saturday and Sunday, more noticably for casual riders. This suggests that both members and casual riders take longer rides over weekends, but particularly casual riders. However, the peak may be more distinct for casual riders as members tend to use Cyclistic services throughout the week whereas casual riders are less likely to do so. 
<br><br<br>


#### Peak Months of Use and Average Ride Lengths: Members vs Casual:

```sql
SELECT month_name,
       member_casual,
       COUNT(DISTINCT ride_id) AS rides,
       ROUND(AVG(ride_length_mins), 2) AS avg_ride_length_mins
FROM trips_2025_clean
GROUP BY month_name, member_casual
ORDER BY month_name, member_casual;
```

<img width="500" height="400" alt="Monthly stats" src="https://github.com/user-attachments/assets/dc99a008-4f76-401c-9a43-12d28ce0f251" />

<br>
The results showed the peak months: <br>

*Member* - August, September, July <br>
*Casual user* - August, July, September
<br><br>
This demonstrates that both members and casual riders use Cyclistic mostly during the summer months: July, August and September, which is to be expected due to better weather conditions. 
<br><br>
It also shows that average ride lengths for both members and casual riders were longer during the warmer months (spring, summer, early autumn).
<br><br><br>


#### Most used start stations & their average ride lengths - members vs casual users

```sql
--members:
SELECT start_station_name,
       COUNT(DISTINCT ride_id) AS rides,
       AVG(ride_length_mins) AS avg_ride_length_mins,
       AVG(start_lat) AS start_lat,
       AVG(start_lng) AS start_lng  ##some stations had a slightly varying lat/long for different rides so I averaged them
FROM trips_2025_clean
WHERE start_station_name IS NOT NULL
  AND start_station_name <> ''
  AND start_lat IS NOT NULL
  AND start_lat <> ''
  AND start_lng IS NOT NULL
  AND start_lng <> ''
  AND member_casual = 'member'
GROUP BY start_station_name;

--casual riders:
SELECT start_station_name,
       COUNT(DISTINCT ride_id) AS rides,
       AVG(ride_length_mins) AS avg_ride_length_mins,
       AVG(start_lat) AS start_lat,
       AVG(start_lng) AS start_lng  ##some stations had a slightly varying lat/long for different rides so I averaged them
FROM trips_2025_clean
WHERE start_station_name IS NOT NULL
  AND start_station_name <> ''
  AND start_lat IS NOT NULL
  AND start_lat <> ''
  AND start_lng IS NOT NULL
  AND start_lng <> ''
  AND member_casual = 'casual'
GROUP BY start_station_name;
```

<img width="400" height="300" alt="top 10 stations - members" src="https://github.com/user-attachments/assets/8ba3b872-af3e-416b-a200-ea4c46ddd5dd" /> <img width="400" height="300" alt="top 10 stations - casual riders" src="https://github.com/user-attachments/assets/2329e8bd-486a-4a0a-9cac-054885da6ee4" />

<br><br><br>


## Conclusions and Recommendations

<br><br>
#### 1. How do annual members and casual riders use Cyclistic bikes differently?
<br>

**Summary**
<br>

<img width="627" height="499" alt="member vs casual summary" src="https://github.com/user-attachments/assets/13462025-36b7-4e71-8ab9-ea311fba53e7" />


<br><br>
#### 2. Why would casual riders buy Cyclistic annual memberships?
- Casual members possibly visitors/tourists to Chicago. Therefore expand nationwide to other cities to entice casual riders living throughout North America to become a member. 
<br><br>
- Casual members possibly commute through other means (driving, public transport etc.). Therefore advertise and emphasise the benefits of Cyclistic, healthy lifestyles, convenience, saving costs. 
<br><br>
- Introducing tourist/seasonal memberships. 
<br><br>
- Create membership offers where longer ride durations are cheaper to entice casual members who tend to ride for longer periods of time. 
<br>

#### 3. How can Cyclistic use digital media to influence casual riders to become members?
- Target and time social media and online advertisement during commuting and rush hours. 
<br><br>
- Offline advertisements of membership offers at bus stops, train stations and other commuting hubs. 
<br><br>
- Offline Advertisements of membership offers at tourist hotspots and scenic areas where casual members are likely to be riding. 
<br><br>
- Analyse certain days likely with heavier traffic/commuting problems and target adverts accordingly. 
<br><br>
- Advertise memberships/membership offers in peak seasons (warmer months) and weekends. 
<br><br>
- Collect gender and age data of users. With this data it will be possible to analyse and gain insights such as whether a specific age group prefer memberships or casual rides. Can also determine if students use Cyclistic services for commuting and offer student memberships.
