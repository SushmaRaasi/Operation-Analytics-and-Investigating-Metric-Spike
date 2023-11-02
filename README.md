# Operation-Analytics-and-Investigating-Metric-Spike

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

### Case Study 1: Job Data Analysis
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
<b>1) Jobs Reviewed Over Time:</b><br>
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

 <b>2) Throughput:</b> <br>
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

<b>3)Language Share Analysis:</b>
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
  <i>Distribution of Percentage Share of Each Language:</i><li>A donut chart visually represents the distribution of percentage share for each language, emphasizing the dominant languages and revealing potential opportunities for targeted marketing efforts.</li>
  I image Donut chart
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
<b>4)Duplicate rows Detection:</b>
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

### Case Study 2: Investigating Metric Spike
#### Dataset 

[Users DataSet](https://docs.google.com/spreadsheets/d/1NY9W5fW2DU3Db5uEAi0allOk9G8M-hxHokZ3CLpjQbA/edit#gid=492277679)
### Process
### Conclusion
