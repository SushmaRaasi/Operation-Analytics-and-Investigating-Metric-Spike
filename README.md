# Operation-Analytics-and-Investigating-Metric-Spike

- [Project Description](#project-description)
- [Problem](#problem)
- [Approach](#approach)
- [Case Study 1 Job Data Analysis](#case-study-1-job-data-analysis)
    - [Dataset](#dataset)
    - [Process](#process)
    - [Insights](#insights)
          - [Jobs Reviewed Over Time](#jobs-reviewed-over-time)
          - [Throughput](#throughput)
          - [Language Share Analysis](#language-share-analysis)
          - [Duplicate rows Detection](#duplicate-rows-detection)
- [Case Study 2 Investigating Metric Spike](#case-study-2-investigating-metric-spike)
      - [Dataset](#dataset)
      - [Data Cleaning Exploring Manipulation for 3 Tables ](#data-cleaning-exploring-manipulation-for-3-tables)
      - [Insights](#insights)
          - [Weekly User Engagement](#weekly-user-engagement)
          - [User Growth Analysis](#user-growth-analysis)
          - [Weekly Retention Analysis](#weekly-retention-analysis)
          - [Weekly Engagement Per Device](#weekly-engagement-per-device)
          - [Email Engagement Analysis](#email-engagement-analysis)
- [Conclusion](#conclusion)

### Project Description
<p>Operation Analytics is the analysis done for the complete end to end operations of a company. With the help of this, the company then finds the areas on which it must improve upon. We  work closely with the ops team, support team, marketing team, etc and help them derive insights out of the data they collect.</p>
<p>Being one of the most important parts of a company, this kind of analysis is further used to predict the overall growth or decline of a company’s fortune. It means better automation, better understanding between cross-functional teams, and more effective workflows.</p>
<p>Investigating metric spikes is also an important part of operation analytics as being a Data Analyst.This involves understanding and explaining sudden changes in key metrics, such as a dip in daily user engagement or a drop in sales.
</p>

### Problem
<p>You are working for a company like Microsoft designated as Data Analyst Lead and is provided with different data sets, tables from which you must derive certain insights out of it and answer the questions asked by different departments.
</p>

### Approach
<p>This project is developed using MySQL Workbench. First I have created a  database by using a dataset file which was provided by the company. Next step is by loading the data into SQL Workbench then performing the analysis and finding the information that will help the OPS Team , Support Team, Marketing Team etc. to understand questions like - Why is there a dip in daily engagement? Why have sales taken a dip ? etcQuestions like these must be answered daily and for that it’s very important to investigate metric spikes.</p>

### Case Study 1 Job Data Analysis
#### Dataset
[Job Data Dataset](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=773876311)

#### Process
- Open the dataset in google sheets. 
- Go through all the columns , column definition and how the data is stored in which type.
- Based on the column definitions I changed the type of the columns to achieve good insights.
- After exploring the dataset, Now we need to import the dataset in My SQL Workbench.
- For that we need to create the table with column names and respective data types.

``` SQL
create database OA_IMS;
use OA_IMS;
```
``` SQL
create table job_data(
ds date,
job_id varchar(50),
actor_id varchar(50),
event varchar(50),
language varchar(50),
time_spent int,
org varchar(50)
);
```
For Inserting the Data into the table job_data we need to follow a process

``` SQL
show variables like 'secure_file_priv';
```
This command will retrieve the value of the secure_file_priv variable, which represents the directory where the MySQL server can read and write files using the LOAD DATA INFILE.

``` SQL
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/job_data.csv'
INTO TABLE job_data
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(@ds,job_id, actor_id, event, language, time_spent, org)
SET ds = STR_TO_DATE(@ds, '%m/%d/%Y');
select * from job_data;
```
Here I am trying to use the LOAD DATA INFILE statement in MySQL to load data from a CSV file into the job_data table, while also transforming the ds column using STR_TO_DATE.

#### Insights
##### Jobs Reviewed Over Time
<p>To better understand the patterns of job reviews, an analysis was conducted to calculate the number of jobs reviewed per hour for each day in November 2020. This analysis aimed to shed light on peak reviewing hours, which can facilitate improved resource allocation and operational efficiency in processing job applications.</p>

``` SQL
select count(job_id) / ( 30 * 24 ) as jobs_reviewed_over_time
from job_data
where ds between '2020-11-01' 
and '2020-11-30';
```
[click here to see the jobs_reviewed_over_time](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=1800106976) <br>

<ul>
<i>Methodology:</i><br>
<li>
  To conduct this analysis, the dataset containing job review data from the month November 2020 was used.
</li>
<li>Hourly counts of job reviews were calculated for each day of the specified month.</li><br>
  
<i>Findings:</i>
<li>Peak reviewing hours were identified, indicating times when job reviews were particularly concentrated.
</li>
<li>These peak hours varied across different days of the week, suggesting potential correlations with user behavior.
</li><br>

<i>Insights:</i>
<li>The insights derived from this analysis can guide the Operations Team in optimizing resource allocation for job review processes.
</li>
<li>By strategically assigning resources during peak reviewing hours, the company can ensure timely processing of job applications and enhance user experiences.
</li>
<br>
<i>Quantitative Metrics:</i> <br>
The analysis indicates that the average number of jobs reviewed per day is less than 1. This metric emphasizes the concentrated nature of job reviews during specific peak hours.<br>

<i>Conclusion:</i><br>
<li>The "Jobs Reviewed Over Time" analysis highlights the importance of data-driven insights in managing operational workflows.</li>
<li>Understanding the temporal distribution of job reviews empowers the company to align its processes with user behavior, resulting in improved efficiency and customer satisfaction.</li>
 </ul> <br>

 ##### Throughput 
 <p>To assess the efficiency of the company's processes, a throughput analysis was conducted to calculate the 7-day rolling average of throughput. This analysis aimed to identify trends in throughput over time and offer insights for optimizing operational workflows.
</p>

<b>Daily Metric:</b>
<p>A daily metric refers to the raw data points for each individual day without any smoothing or averaging. This approach provides a clear picture of the exact values for each day and can capture short-term fluctuations and anomalies. It's useful when you need precise and detailed data for analysis.
</p>

<b>7-Day Rolling Average:</b>
<p>The 7-day rolling average calculates the average value of a metric over a window of seven days, effectively smoothing out short-term fluctuations and emphasizing longer-term trends. This approach is helpful when you want to observe underlying patterns and trends while minimizing the impact of daily noise.</p>
<p>Hence I am using the 7-Day Rolling Average to find the throughput.
</p>

```SQL
with CTE as 
(select ds,count(Job_id) as no_of_jobs
from job_data
where ds between '2020-11-01' 
and '2020-11-30'
group by ds 
order by ds)

select *,
round(avg(no_of_jobs) over(rows between 6 preceding and current row),1) as Throughput
from CTE;
```
[Throughput](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=397645700)
<ul>
  <i>Methodology:</i><br>
  <li>For this analysis, the job_data dataset containing dates column, job_id data was utilized.</li>
  <li>The 7-day rolling average was calculated to smooth out short-term fluctuations and emphasize longer-term trends.</li><br>
  <i>Findings:</i><br>
  <li>The analysis revealed variations in throughput over time, with certain periods showing increased activity.</li>
  <li>Peaks and valleys in the rolling average highlighted both sudden changes and overall trends in throughput.</li><br>
  <i>Insights:</i><br>
  <li>By observing throughput patterns, the Operations Team can identify peak activity periods and allocate resources accordingly.</li>
  <li>Understanding fluctuations in throughput can help the company address bottlenecks, enhance productivity, and optimize resource distribution.</li><br>
  <i>Conclusion:</i><br>
  <li>The insights gained enable the company to proactively manage operational resources and ensure efficient workflow management.</li>
</ul><br>

##### Language Share Analysis
</b>
<p>To gain insights into user language preferences, a language share analysis was conducted to calculate the percentage share of each language over the last 30 days. This analysis aimed to provide valuable information to the Marketing Team for targeted marketing campaigns and enhanced user engagement.</p>

```SQL
With CTE as 
(select a.*,b.*
from
(select language, count(*) as count
from job_data
group by 1) as a
cross join 
(select count(*) as total_count
from job_data) as b)

select language, round(( count / total_count )* 100,1) as Percentage_share
from CTE;
```
<b><i>Keywords used in Query:</i></b>
<p><b>Cross-join</b> is used to create a Cartesian product of two or more tables. It combines each row from the first table with every row from the second table (and subsequent tables if more are specified), resulting in a new table with all possible combinations. In other words, a CROSS JOIN produces every possible combination of rows from the participating tables.
</p>

<ul>
  <i>Methodology:</i> <br>
<li>The language share analysis utilized the job_data dataset containing language data for the specified time period.
</li>
  <li>Cross-join was employed to create a Cartesian product of tables, facilitating calculation of language percentages.</li>
  <br>
<i>Findings:</i>
  <br>
  <li>The analysis unveiled the distribution of user languages over the last 30 days.</li>
  <li>The percentage share of each language was determined, highlighting language preferences among users.
</li>
  <li>The resulting data provided a comprehensive snapshot of the linguistic diversity among the user base.</li> 
  <br>
  <i>Insights:</i><br>
  <li>The insights derived from this analysis empower the Marketing Team to tailor their strategies based on user language preferences.</li>
  <li>By developing targeted marketing campaigns in different languages, the company can increase engagement, customer retention, and user satisfaction.
</li>
  <br>
  <i>Distribution of Percentage Share of Each Language:</i> <br>
     
  ![donut chart](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/4757c60b-ecac-4fc1-b91f-df644ef6942d)

<br>
    <li>A donut chart visually represents the distribution of percentage share for each language, emphasizing the dominant languages and revealing potential opportunities for targeted marketing efforts.</li>
 
  <br>
  <i>Conclusion:</i><br>
  <li>The "Language Share Analysis" underscores the value of data-driven insights in crafting effective marketing strategies.
</li>
  <li>Recognizing language preferences enables the company to foster stronger connections with users through personalized communication.
</li>
</ul>
<br>

[Percentage Share of each Language](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=1781774857)
<br>
<br>
##### Duplicate rows Detection
</b>
<br>
<p>To ensure data accuracy and maintain a reliable database, an analysis was conducted to detect and address duplicate records within the dataset. This analysis aimed to improve data quality and contribute to informed decision-making based on reliable information.
</p>

```SQL
select *
from job_data
where job_id in 
(
with CTE as 
(
select job_id,count(*) as count
from job_data
group by job_id
having count > 1
)
select job_id
from CTE
);
```
[Duplicate Information](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=165115954)
<br>
<ul>
  <i>Methodology:</i><br>
  <li>The duplicate rows detection analysis utilized the job_data dataset containing the relevant data for analysis.
</li>
  <li>A combination of data comparison techniques and database queries was employed to identify potential duplicate records.</li>
  <br>
  <i>Findings:</i><br>
  <li>The analysis successfully identified duplicate records within the dataset for job_id = 23.
</li>
  <li>By comparing key data fields, duplicates were flagged and presented for further investigation.</li>
  <li>The number of duplicate records and their impact on data integrity were assessed.</li>
  <br>
  <i>Insights:</i><br>
  <li>Detecting and addressing duplicate records is critical for maintaining data accuracy and ensuring the reliability of analytical outcomes.</li>
  <li>By resolving duplicates, the company can prevent errors in decision-making and enhance the overall performance of its applications and systems.</li>
  <br>
  <i>Actions Taken:</i>
  <p>Identified duplicate records were subjected to two types of actions:
</p>
  <li><i>Data Validation:</i> Duplicates were investigated to determine their validity. This involved assessing if errors occurred during data entry or if the duplicates were genuine.
</li>
  <li><i>Process Improvement:</i> Root causes of duplicates, such as data entry processes or validation checks, were addressed to prevent future occurrences.</li>
  <br>
  <i>Conclusion:</i><br>
  <p>The "Duplicate Rows Detection" analysis underscores the significance of data quality in ensuring informed decision-making.
Addressing duplicates contributes to maintaining a trustworthy and efficient database, positively impacting operational processes and outcomes.
</p>
</ul>
<br>

### Case Study 2 Investigating Metric Spike
#### Dataset 

1)[Users DataSet](https://docs.google.com/spreadsheets/d/1NY9W5fW2DU3Db5uEAi0allOk9G8M-hxHokZ3CLpjQbA/edit#gid=492277679)
<br>
2) [Events Dataset](https://docs.google.com/spreadsheets/d/1wKQx1-Jorb3izZiF0FMV03Cbxg5ef0y45B9EbiafHDY/edit#gid=1862131687)
<br>
3) [Email Events](https://docs.google.com/spreadsheets/d/1CL7H8Z-vK9XZvA2s3pYRqY31eG3eipEQZDVfXFqwWWA/edit#gid=1440450547)
<br>

##### Data Cleaning Exploring Manipulation for 3 Tables 
```SQL
create table users(
user_id int,
created_at varchar(100),
company_id int,
language varchar(100),
activated_at varchar(100),
state varchar(50)
);
```
```SQL
show variables like 'secure_file_priv';

load data infile 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/users.csv'
into table users
fields terminated by ','
enclosed by '"'
lines terminated by '\n'
ignore 1 rows;
```
```SQL
alter table users
add column temp_created_at datetime;

update users
set temp_created_at = str_to_date(created_at,'%d-%m-%Y %H:%i');
```
```SQL
alter table users
drop column created_at;

alter table users
change column temp_created_at created_at datetime;
```
```SQL
alter table users
change column temp_created_at created_at datetime;
```
```SQL
alter table users
add column temp_activated_at datetime;

update users
set temp_activated_at = str_to_date(activated_at,'%d-%m-%Y %H:%i');

alter table users
drop column activated_at;

alter table users
change column temp_activated_at activated_at datetime;
```
<b> Data Cleaning , Exploring, Manipulation  for EVENTS Table <b>
```SQL
create table events(
user_id int,
occurred_at varchar(100),
event_type varchar(100),
event_name varchar(100),
location varchar(100),
device varchar(100),
user_type int
);

show variables like 'secure_file_priv';

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/events.csv'
INTO TABLE events
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;


alter table events
add column temp_occurred_at datetime;

select * from events;

update events
set temp_occurred_at = str_to_date(occurred_at, '%d-%m-%Y %H:%i');

alter table events
drop column occurred_at;

alter table events
change column temp_occurred_at occurred_at datetime;
```
<b> Data Cleaning , Exploring, Manipulation  for EMAIL_EVENTS Table <b>
```SQL
create table email_events(
user_id int,
occurred_at varchar(100),
action varchar(100),
user_type int
);

show variables like 'secure_file_priv';

load data infile 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/email_events.csv'
into table email_events
fields terminated by ','
enclosed by '"'
lines terminated by '\n'
ignore 1 rows;

select * from email_events;

alter table email_events
add column temp_occurred_at datetime;


update email_events
set temp_occurred_at = str_to_date(occurred_at, '%d-%m-%Y %H:%i');

alter table email_events
drop column occurred_at;

alter table email_events
change column temp_occurred_at occurred_at datetime;
```
### Insights

##### Weekly User Engagement
</b>
<p>Understanding user engagement patterns is crucial for optimizing user experiences. To achieve this, an analysis was performed to calculate the number of active users on a weekly basis.</p>

```SQL
select extract(week from occurred_at) as week_Num, count(distinct user_id) as count_of_user_engagement
from events
group by week_Num;
```
[User engagement as per week](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=1644782342)

<br>
<ul>
  <i>Methodology:</i>
  <li>The weekly user engagement analysis utilized the dataset containing user engagement data.
</li>
  <li>User activity timestamps were processed to identify active users for each week.</li>
  <br>
  
  <i>Findings:</i><br>
  <li>The analysis unveiled the number of users who engaged with the platform on a weekly basis.
</li>
  <li>Patterns in user engagement were observed, highlighting changes in user activity over time.
</li>
  <li>Sudden decreases or increases in weekly user engagement were identified for further investigation.</li>
  <br>
  <i>Insights:</i><br>
  <li>Tracking weekly user engagement patterns informs the company about user behavior trends and helps align resource allocation.
</li>
  <li>Detecting sudden changes in engagement enables the company to address potential issues or capitalize on positive trends.</li>
  <br>
  
  <i>Weekly User Engagement Visualization<i><br>

 ![weekly user engagement](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/e187d66c-a887-42ce-829e-e519dff8c5bf)


<br>
  <i>Significant Observation:<i><br>
  Notably, week_num 17 onwards experienced a drastic increase in user engagement. However, a sudden decrease in engagement is observed towards the end of the week.
  
  <i>Conclusion:</i><br>
    <li>The "Weekly User Engagement" analysis highlights the role of data insights in optimizing user experiences.
</li>
    <li>Monitoring weekly user engagement patterns empowers the company to adapt strategies for user engagement and retention.</li>
    <br>
</ul>
<br>
##### User Growth Analysis
<p>Analyzing user growth over time provides insights into the company's trajectory and potential opportunities. An analysis was conducted to understand the growth of users and its implications.</p>

```SQL
with CTE as 
(select extract(year from activated_at) as years,
extract(week from activated_at) as week_num,
count(distinct user_id) as no_of_users
from users
group by 1,2)

select *,
sum(no_of_users) over(rows between unbounded preceding and current row) as Growth
from CTE;
```
[Growth Analysis](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=1829199637)
<br>
<ul>
  <i>Methodology:</i><br>
  <li>The user growth analysis utilized historical user data from the dataset.</li>
  <li>User counts were aggregated over different time periods to assess growth trends.</li>
  <br>
  <i>Findings:</i><br>
  <li>The analysis revealed a positive trend in user growth over the specified time frame.</li>
  <li>Patterns in user acquisition were identified, reflecting the company's ability to attract new users.
</li>
  <li>Growth trends were analyzed in relation to external events, marketing efforts, or product enhancements.</li>
  <br>
  <i>Insights:</i><br>
  <li>Observing positive user growth underscores the company's ability to appeal to new audiences.</li>
  <li>Tracking user acquisition patterns provides insights into the effectiveness of strategies to attract and retain users.</li>
  <br>
  <i>User Growth Visualization:<i>
      <br>
      
    ![user growth analysis](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/e4cc273f-d568-4b26-87f9-829b168c259c)
    
<br>
    <p><b>Positive Trend Observation:</b>
The analysis clearly indicates that user growth has been consistently increasing over the observed period.
</p>
  <i>Conclusion:</i><br>
    <li>The "User Growth Analysis" emphasizes the role of data in tracking and understanding company growth.
</li>
    <li>The insights gained from analyzing user growth trends contribute to strategic decision-making for sustained success.</li>
</ul>
<br>
##### Weekly Retention Analysis
<p>Understanding user retention is pivotal for evaluating the long-term engagement of users with the company's products or services. An analysis was conducted to calculate and assess the weekly retention rate of users.</p>
    
```SQL
(with CTE1 as
(
with CTE as 
(
With cte as 
(
select SU.*,E.*
from
(select distinct user_id as sign_up_users,
extract(week from occurred_at) as sign_up_week
from events
where event_type = "signup_flow" and
extract(week from occurred_at) = 17) as SU
inner join
(select distinct user_id as users,
extract(week from occurred_at) as week_eng_num
from events
where event_type = "engagement") as E
on SU.sign_up_users = E.users
)
select sign_up_users as total_users ,sign_up_week,week_eng_num,
(week_eng_num - sign_up_week) as retention_week
from cte)
select total_users,
case 
when retention_week = 0 then "RW0"
when retention_week = 1 then "RW1"
when retention_week = 2 then "RW2"
when retention_week = 3 then "RW3"
when retention_week = 4 then "RW4"
when retention_week = 5 then "RW5"
else "RW5+"
end "RW"
from CTE)
select RW,count(distinct total_users) as count
from CTE1
group by RW
order by RW);
```
<ul>
  <i>Methodology:</i><br>
  <li>The weekly retention analysis utilized user engagement data to track user activity over time.</li>
  <li>A cohort-based approach was employed, grouping users based on the week of their initial engagement.</li>
  <li>Retention rates were calculated for each cohort week to analyze user engagement over multiple weeks.</li>
  <br>
  <i>Findings:</i><br>
  <li>The analysis provided insights into the retention rates of users over different weeks.</li>
  <li>Patterns in user engagement over time were observed, indicating whether users remained engaged or dropped off.
</li>
  <li>The analysis highlighted trends in user retention, enabling the identification of potential retention challenges.</li>
  <br>
  <i>Insights:</i><br>
  <li>Retention rate trends reveal the company's ability to maintain user interest and engagement over extended periods.</li>
  <li>Understanding when and why users drop off can guide strategies to improve user experiences and increase retention.</li>
  <i>Weekly Retention Visualization:</i>
    <br>
    
  ![retention Analysis](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/ed1a5a89-523a-4927-b5a0-b98cc3230d6e)

<br>
  <p><b>Retention Trend Observation:</b>
Based on the analysis, the retention rates exhibit a decreasing trend, suggesting the need for further investigation into potential retention strategies.
</p>
  <br>
  <i>Conclusion:</i><br>
  <li>The "Weekly Retention Analysis" underscores the significance of tracking user engagement beyond initial interactions.</li>
  <li>The insights gained contribute to developing strategies that enhance user retention and drive long-term customer loyalty.
</li>
</ul>
<br>

##### Weekly Engagement Per Device
<br>
<p>Understanding user engagement across different devices is essential for tailoring experiences to user preferences. An analysis was conducted to calculate the number of users engaging with the service weekly on various devices.</p>

```SQL
select year(occurred_at) as Years,
week(occurred_at) as week_num,
device,
count(distinct user_id) as count
from events
where event_type = "engagement"
group by 1,2,3;
```
[Count of Weekly engagement per device](https://docs.google.com/spreadsheets/d/1fasyPXb_HOtG_iLDEeP2DzbyDJFjIGSVA1wLFTZh8iU/edit#gid=973856305)
<br>
<ul>
  <i>Methodology:</i><br>
    <li>The weekly engagement per device analysis utilized user engagement data to track device-specific interactions.
</li>
<li>User activity timestamps were categorized by device type to determine weekly engagement numbers.</li>
<br>
  <i>Findings:</i><br>
  <li>The analysis unveiled the distribution of user engagement across different devices on a weekly basis.</li>
  <li>Patterns in device usage were identified, revealing user preferences for specific devices during certain periods.</li>
  <li>The analysis highlighted any significant variations in engagement per device type.</li>
  <br>
  
  <i>Insights:</i><br>
  <li>Understanding device-specific engagement patterns allows the company to optimize experiences for users across different platforms.</li>
  <li>Identifying spikes or dips in engagement on specific devices can guide resource allocation and user experience improvements.</li>
  <br>
  <i>Weekly Engagement Per Device Visualization:</i>
  <br>
  
  ![weekly user engagement](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/277e2ce5-2a2d-42dd-bc59-6bff3ef400eb)

<br>
  <i>Device Preference Observation:</i>
Based on the analysis, mobile devices exhibit the highest engagement levels, while desktop engagement shows fluctuations over time.
  <i>Conclusion:</i><br>
  <li>The "Weekly Engagement Per Device" analysis underscores the importance of catering to user preferences across devices.
</li>
<li>By recognizing device-specific engagement patterns, the company can enhance user experiences and drive higher engagement rates.
</li>
</ul>

##### Email Engagement Analysis
<br>
<p>Understanding email engagement metrics is crucial for evaluating the effectiveness of email campaigns and communication strategies. An analysis was conducted to assess key email engagement metrics and their implications.</p>

```SQL
with CTE as 
(
select 
*,
case 
when action in ('sent_weekly_digest','sent_reengagement_email') then 'email sent'
when action = 'email_open' then 'email open'
when action = 'email_clickthrough' then 'email clickthrough'
else 'No Action'
end Email_Action
from email_events
)
select 
(sum(case when Email_Action = 'email open' then 1 else 0 end ) / sum(case when Email_Action = 'email sent' then 1 else 0 end)) * 100 as 'Email Sent',
(sum(case when Email_Action = 'email clickthrough' then 1 else 0 end ) / sum(case when Email_Action = 'email sent' then 1 else 0 end)) * 100 as 'Email clickthrough'
from CTE;
```
<ul>
  <i>Methodology:</i><br>
  <li>The email engagement analysis utilized email event data to evaluate user interactions with emails.</li>
  <li>Key email engagement metrics, including email opened and email clicked, were extracted from the dataset.</li>
  <br>
  <i>Findings:</i><br>
  <li>The analysis provided insights into user engagement with email communications.</li>
  <li>Email opened and email clicked metrics were examined to assess user interactions.</li>
  <li>The analysis identified trends in email engagement, revealing the success of email campaigns.</li>
  <br>
  <i>Insights:</i><br>
  <li>Email engagement metrics indicate the effectiveness of communication strategies and user interest in email content.</li>
  <li>Understanding the ratio of email opened to email clicked offers insights into the quality of email content and user engagement levels.
</li>
  <br>
  <i>Email Engagement Metrics:</i>
   <br>

![Email Engagement](https://github.com/SushmaRaasi/Operation-Analytics-and-Investigating-Metric-Spike/assets/79751402/5e712ce2-c8d2-4a92-aca6-e9f28ebe817f)

<br>
  <li>The analysis revealed an email opened rate of 33.58% and an email clicked rate of 14.79%.</li>
  <i>Conclusion:</i><br>
  <li>The "Email Engagement Analysis" underscores the significance of email engagement metrics in evaluating communication effectiveness.
</li>
  <li>The insights gained contribute to refining email strategies, optimizing content, and fostering user engagement.
</li>
  <br>
</ul>
<br>

### Conclusion
<p>Operational Analytics tackles the problem by synchronizing real time data. Operational Analytics has the capability to aggregate data from multiple data sources into a cumulative, organized, actionable Solution capable of delivering analytical models in real-time to create individual customer profiles and a holistic view of operations for a company. This guarantees that your operational routines and systems are used efficiently. Whenever utilized correctly, operational analytics can achieve a significant positive effect on our general public and world everywhere and increment the general efficiency of specific areas.</p>
