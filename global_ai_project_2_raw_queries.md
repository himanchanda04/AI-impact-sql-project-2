
# Global AI Content Impact Dataset - SQL Project (Raw Query Documentation)

## Project Description
This markdown documents the SQL queries used in the second phase of analysis and cleaning for the Global AI Content Impact Dataset. It covers all steps from data preparation and cleaning to feature creation and ranking.

---

## SQL Workflow

```sql
CREATE TABLE global_ai_content_impact_dataset_copy AS
SELECT * FROM global_ai_content_impact_dataset;
select *
from global_ai_content_impact_dataset_copy;
-- finding missing or Null values
SELECT * 
FROM global_ai_content_impact_dataset_copy 
WHERE `AI Adoption Rate (%)` IS NULL 
   OR `Regulation Status` IS NULL;

-- standardize the data 

UPDATE global_ai_content_impact_dataset_copy
SET Industry = TRIM(UPPER(Industry));

-- outlier detection
SELECT *
FROM global_ai_content_impact_dataset_copy
WHERE `AI Adoption Rate (%)` > 100 
or`Revenue Increase Due to AI (%)` > 100;

-- remove duplicates
WITH CTE AS(	
    select *, row_number() over (partition by country, year, industry order by id) as new_row
	from global_ai_content_impact_dataset_copy)
    
    DELETE FROM global_ai_content_impact_dataset_copy
    WHERE id IN(
    Select id
    from cte
    where new_row > 1);
    select country, year, industry, count(*) as cnt
    from global_ai_content_impact_dataset_copy
    GROUP BY country, year, industry
    Having cnt > 1;
    
-- which countries have the most job loss due to AI %
select *
from global_ai_content_impact_dataset_copy;

select Distinct country, `Job Loss Due to AI (%)` as Job_loss
from global_ai_content_impact_dataset_copy
order by `Job Loss Due to AI (%)` desc;
select country, count(*) as job_loss_country
from global_ai_content_impact_dataset_copy
group by country
order by job_loss_country desc;

-- which industry hit by most with AI adoption result in jobloss
select distinct industry, count(`Job Loss Due to AI (%)`) as job_loss
from global_ai_content_impact_dataset_copy
group by industry
order by job_loss desc
limit 10;

Alter table global_ai_content_impact_dataset_copy
Add column job_loss Float;

Update global_ai_content_impact_dataset_copy AS Main
Join (
select distinct industry, count(`Job Loss Due to AI (%)`) as job_loss
from global_ai_content_impact_dataset_copy
group by industry) as Sub
On Main.Industry = Sub.Industry
Set Main.job_loss = Sub. job_loss;

Select *
From global_ai_content_impact_dataset_copy;

-- industry has most AI adoption rate %
Select *
from global_ai_content_impact_dataset_copy;

select Distinct Industry, `AI Adoption Rate (%)`
from global_ai_content_impact_dataset_copy
order by `AI Adoption Rate (%)` desc;

-- Average AI adoption rate by industry
select Distinct Industry, 
Round(Avg(`AI Adoption Rate (%)`), 0) avg_adoption
from global_ai_content_impact_dataset_copy
group by industry
order by avg_adoption desc;

Alter Table global_ai_content_impact_dataset_copy
Add column avg_adoption FLOAT;

Update global_ai_content_impact_dataset_copy Main
Join 
	(select Distinct Industry, 
	Round(Avg(`AI Adoption Rate (%)`), 0) As avg_adoption
	from global_ai_content_impact_dataset_copy
	group by industry) as sub
On Main.Industry = sub.Industry
set  Main.avg_adoption = sub.avg_adoption;

select industry, `AI Adoption Rate (%)`, avg_adoption
from global_ai_content_impact_dataset_copy;

Alter Table global_ai_content_impact_dataset_copy
drop column avg_ai_adoption_by_industry
;
-- Highest Revenue increase due to AI in which industry
Select *
from global_ai_content_impact_dataset_copy;

select Distinct industry, Max(`Revenue Increase Due to AI (%)`) as max_revenue
from global_ai_content_impact_dataset_copy
group by industry
order by max_revenue desc
;
Alter Table global_ai_content_impact_dataset_copy
Add column max_revenue float;
Update global_ai_content_impact_dataset_copy AS Main
Join (
		select Distinct industry, Max(`Revenue Increase Due to AI (%)`) as max_revenue
		from global_ai_content_impact_dataset_copy
		group by industry
	) as sub
On Main. Industry = sub. Industry
Set Main.max_revenue = sub.max_revenue; 

Select *
from global_ai_content_impact_dataset_copy;

-- group by industry in regulation status
Select *
from global_ai_content_impact_dataset_copy;

Select Distinct Industry,
CASE
	when `Regulation Status` = 'Strict' Then 'Very Good Governance'
    when `Regulation Status` = 'Moderate' Then 'Good Governance'
	when `Regulation Status` = 'Linient' Then 'Bad Governamce'
	Else 'Very Bad'
End as Reg_status,
Count(*) as count_per_group
from global_ai_content_impact_dataset_copy
Group by Industry,Reg_status 
order by Industry;

-- To check which country has the best governance and which one has the worst
-- or the average governance
Select Distinct country, year, `Regulation Status`, Round(Avg(
CASE
	when `Regulation Status` = 'Strict' Then 1
    when `Regulation Status` = 'Moderate' Then 2
	when `Regulation Status` = 'Linient' Then 3
	Else 4
End), 2) as Avg_governance_score
from global_ai_content_impact_dataset_copy
Group by country,year, `Regulation Status` 
order by country,year, Avg_governance_score asc;

Alter Table global_ai_content_impact_dataset_copy
Add column Avg_governance_score float;
Update global_ai_content_impact_dataset_copy as main
Join 
( Select Distinct country, `Regulation Status`, Round(Avg(
CASE
	when `Regulation Status` = 'Strict' Then 1
    when `Regulation Status` = 'Moderate' Then 2
	when `Regulation Status` = 'Linient' Then 3
	Else 4
End), 2) as Avg_governance_score
from global_ai_content_impact_dataset_copy
Group by country) as sub
	On main.country = sub.country
    set main.Avg_governance_score = sub. Avg_governance_score;

-- we need to group by industries and regulations
select country,
Group_concat(Distinct industry order by industry separator ',') as Industries,
Group_concat(Distinct `Regulation Status` order by `Regulation Status` separator ',') as regulation_statuses
from global_ai_content_impact_dataset_copy
group by country
order by country;

select Distinct country, year,
industry,
`Regulation Status`
from global_ai_content_impact_dataset_copy
group by country,industry, year, `Regulation Status`
order by country, year asc;

-- -- How Many Industries Are Strict/Moderate/Lenient per Country
select  `Regulation Status`,
count(distinct country) as country_count
from global_ai_content_impact_dataset_copy
group by `Regulation Status`
;
-- answer showing is wrong, so we need to try different query for countries with multiple regulations types
select country, 
count(Distinct `Regulation Status`) as country_status
from global_ai_content_impact_dataset_copy
group by country
order by country_status asc;

select country,max(max_revenue) as highest_revenue
from global_ai_content_impact_dataset_copy
group by country
order by highest_revenue desc
limit 5 ;
-- find rank and dense rank in this table
select country, MAX(max_revenue) as max_rev,
rank() over (order by MAX(max_revenue) asc) as revenue_rank,
DENSE_RANK () over (order by MAX(max_revenue) asc) as revenue_dense_rank
from global_ai_content_impact_dataset_copy
GROUP BY COUNTRY;

ALTER TABLE global_ai_content_impact_dataset_copy
ADD COLUMN revenue_rank FLOAT
;
UPDATE global_ai_content_impact_dataset_copy AS MAIN
JOIN (
	select id,
	rank() over (order by round(max_revenue, 2) asc) as revenue_rank,
	DENSE_RANK () over (order by round(max_revenue, 2) asc) as revenue_dense_rank
from global_ai_content_impact_dataset_copy
) AS SUB
	ON MAIN.id = sub.id
    SET MAIN.revenue_rank = sub. revenue_rank,
		MAIN.revenue_dense_rank = SUB.revenue_dense_rank ;
        
select count(distinct max_revenue) as unique_revenue_count
from global_ai_content_impact_dataset_copy;
    
select *
from global_ai_content_impact_dataset_copy
order by revenue_dense_rank asc;
;
alter table global_ai_content_impact_dataset_copy
drop column rev_rank;

```

---

## Notes
- All queries were executed using MySQL.
- The project follows a step-by-step process for cleaning, transformation, and analysis.
- Raw SQL queries are documented here for transparency and reproducibility.

## Author
Himanshu Manchanda  
LinkedIn: https://www.linkedin.com/in/himanshuman7  
Location: Winnipeg, Canada
