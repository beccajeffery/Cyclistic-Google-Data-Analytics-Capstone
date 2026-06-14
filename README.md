# Cyclistic-Google-Data-Analytics-Capstone
Analysis on historic Cyclistic Bike Share ride data in order to understand the behaviour of casual riders and annual members, with the ultimate goal of designing a new marketing strategy to convert casual riders into annual members.


## Scenario
In this case study for the Google Data Analytics Course Capstone, my role is a junior analyst within the marketing analytics team for a bike-share company Cyclistic in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Consequently, we want to understand how casual riders and annual members use Cyclistic bikes differently. These insights will support a new marketing strategy to convert casual users into annual members. 

Cyclistic is a bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can’t use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each day.


## Ask
The business task is to analyze historical bike trip data to understand the behaviour of casual riders and annual members, with the ultimate goal of designing a new marketing strategy to convert casual riders into annual members. This task is essential for Cyclistic’s growth and success, as it focuses on maximizing annual memberships, which have been identified as more profitable for the company.

Business task: Three questions will guide the future marketing campaign:

  1. How do annual members and casual riders use Cyclistic bikes differently?
  2. Why would casual riders buy Cyclistic annual memberships?
  3. How can Cyclistic use digital media to influence casual riders to become members?

Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.


## Prepare 
For this case study, I used 12 months of Cyclistic’s 2025 bike ride data (01/01/2025 to 31/01/2025) from the following Divvy database: https://divvy-tripdata.s3.amazonaws.com/index.html.
It has been made available by Motivate International Inc. under this license.


IS THE DATA ROCCC?

✅ Reliable - YES, after removing some duplicates and invalid rides, the data is accurate and unbiased. It is also first party data, increasing the reliability. 
✅ Original - YES - the data was downloaded from the original public data source by Motivate LLC.
✅ Comprehensive - YES - despite some missing data, I feel the dataset is extensive and provides more than enough information for effective analysis. Other useful information could include postcodes, gender and ages of members and casual riders, however this would cause privacy concerns.
✅ Current - YES - the data is current and updated on a month basis.
❌ Cited - NO 

### Tools Used
- SQL
- Tableau


## Process
I used MySQL Workbench 8.0 as I had this on my desktop. Here I was able to successfully import the 12 CSV files into MySQL Workbench 8.0 and named them according to the appropriate months (e.g. “01_2025” for January 2025 data, “02_2025” for February 2025 data etc).

I then created a master table called all_trips_2025 that I could later insert the data from the imported monthly tables:

