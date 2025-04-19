
# Global AI Content Impact Dataset - SQL Data Cleaning Project

## Project Overview
This project focuses on cleaning and analyzing the Global AI Content Impact Dataset using SQL. It involves removing duplicates, calculating key metrics like AI adoption and job loss, and evaluating governance based on regulation status.

## Tools Used
- MySQL Workbench
- SQL (CTEs, Window Functions, Aggregates, CASE statements)

## Cleaning Steps
1. Created a duplicate of the original table to avoid altering the raw data.
2. Removed duplicate rows using ROW_NUMBER() with PARTITION BY country, year, industry.
3. Standardized text columns like Industry using TRIM() and UPPER().
4. Checked for missing or NULL values in important columns.
5. Identified and flagged outliers in percentage-based metrics.

## Analysis Performed

### AI Adoption
- Found the industry with the highest AI adoption rate.
- Calculated average AI adoption per industry.
- Stored results in a new column `avg_adoption`.

### Job Loss
- Identified countries and industries with the most job loss due to AI.
- Created a column `job_loss` using COUNT() grouped by industry.

### Revenue Impact
- Found industries with the highest revenue increase due to AI.
- Created a column `max_revenue` per industry.
- Ranked countries by revenue using RANK() and DENSE_RANK().

### Governance
- Categorized regulation status into scores: Strict = 1, Moderate = 2, Linient = 3, Others = 4.
- Calculated average governance score per country and stored it in `avg_governance_score`.

## Final Columns Created
- job_loss
- avg_adoption
- max_revenue
- avg_governance_score
- revenue_rank
- revenue_dense_rank

## Summary
This project uses real-world data to showcase practical SQL skills including data cleaning, transformation, ranking, and feature engineering. It helps demonstrate how AI impacts industries and governance across different countries.

## Author
Himanshu Manchanda  
LinkedIn: https://www.linkedin.com/in/himanshuman7  
Location: Winnipeg, Canada
