# facility_scores
Evaluate the data completeness and timeliness of facilities based on WAIIS vaccination data

Facility Scoring Query

Overview

This query calculates a comprehensive facility score based on data quality and timeliness for vaccination records. It evaluates six critical metrics and assigns a score ranging from 0 to 12 for each facility. High scores indicate excellent data management practices, while low scores highlight areas for improvement.

Features

Filters vaccination records for the year 2024.

Evaluates six key metrics:

Timeliness: Percentage of records entered more than one day after the vaccination date.

Missing VFC Eligibility: Percentage of records with missing VFC eligibility information.

Missing Funding Source: Percentage of records lacking funding source information.

Missing Lot Number: Percentage of records with missing or invalid lot numbers.

Missing Race: Percentage of records without valid race data.

Missing Ethnicity: Percentage of records without valid ethnicity data.

Aggregates metrics and calculates a cumulative score for each facility.

Ranks facilities by scores and identifies those needing support.

Query Structure

1. Filter Vaccination Data

Filters the vaccination_master table to include only records where the vaccination date falls within 2024.

WITH FilteredVaccinationData AS (
    SELECT *
    FROM raw_waiis.vaccination_master
    WHERE YEAR(VACC_DATE) = 2024
)

2. Main Query

Aggregates metrics and calculates scores for each facility:

Facility Information: Includes facility ID and name from facility_master.

Scores: Cumulative score calculated based on defined thresholds for each metric.

Metrics: Includes percentage metrics for timeliness, missing data, and more.

3. Subqueries

Each metric is calculated in a separate subquery and joined with the facility master table:

Timeliness Metrics:

SELECT
    VM.ASIIS_FAC_ID,
    COUNT(*) AS Total_Records,
    ROUND(100.0 * COUNT(CASE WHEN DATEDIFF(DAY, VM.VACC_DATE, VM.INSERT_STAMP) > 1 THEN 1 ELSE NULL END) / COUNT(*), 2) AS Beyond_One_Day_Percent
FROM FilteredVaccinationData AS VM
GROUP BY VM.ASIIS_FAC_ID

Missing Data Metrics: Calculates percentages of missing VFC eligibility, funding source, lot numbers, race, and ethnicity.

4. Final Results

Filters and orders facilities by:

Scores (descending)

Total records (descending)

Metrics percentages (descending)

ORDER BY
    Scores DESC,
    T.Total_Records DESC,
    T.Beyond_One_Day_Percent DESC,
    V.Missing_VFC_Eligible_Percent DESC,
    F.Missing_Funding_Source_Percent DESC,
    L.Missing_Lot_Number_Percent DESC,
    R.Missing_Race_Percent DESC,
    E.Missing_Ethnicity_Percent DESC;

Scoring Criteria

For each metric, thresholds are defined to award points as follows:

2 points: Excellent performance (low percentage of issues).

1 point: Average performance (moderate percentage of issues).

0 points: Needs improvement (high percentage of issues).

Usage

Execute the query in a database supporting SQL syntax (e.g., Databricks, Snowflake).

Replace raw_waiis with the appropriate schema name for your dataset.

Review results to identify high-performing facilities and those requiring intervention.


Contributions

Feel free to fork this repository and submit pull requests for improvements or opt
