
# Facility Scoreing Query

## Overview
This project evaluates the data completeness and timeliness of facilities based on WAIIS vaccination data. The query calculates a comprehensive facility score by assessing six critical metrics, with scores ranging from 0 to 12 for each facility. High scores indicate excellent data management practices, while low scores highlight areas for improvement.

## Features
- **Filters vaccination records based on the vaccination dates.**
- **Evaluates six key metrics:**
  1. **Timeliness:** Percentage of records entered more than one day after the vaccination date.
  2. **Missing VFC Eligibility:** Percentage of records with missing VFC eligibility information.
  3. **Missing Funding Source:** Percentage of records lacking funding source information.
  4. **Missing Lot Number:** Percentage of records with missing or invalid lot numbers.
  5. **Missing Race:** Percentage of records without valid race data.
  6. **Missing Ethnicity:** Percentage of records without valid ethnicity data.
- **Aggregates metrics and calculates a cumulative score for each facility.**
- **Ranks facilities by scores and identifies those needing support.**

## Query Structure

### 1. Filter Vaccination Data
Filters the `vaccination_master` table to include only records where the vaccination date falls within a certain time period.

### 2. Main Query
Aggregates metrics and calculates scores for each facility:
- **Facility Information:** Includes facility ID and name from `facility_master`.
- **Metrics:** Includes percentage metrics for timeliness, missing data, and more.
- **Scores:** Cumulative score calculated based on defined thresholds for each metric.

### 3. Subqueries
Each metric is calculated in a separate subquery and joined with the facility master table:

#### Missing Data Metrics:
Calculates percentages of missing VFC eligibility, funding source, lot numbers, race, and ethnicity.

### 4. Final Results
Filters and orders facilities by:
- **Scores (descending)**
- **Total records (descending)**
- **Metrics percentages (descending)**


## Scoring Criteria
For each metric, thresholds are defined to award points as follows:
- **2 points:** Excellent performance (low percentage of issues).
- **1 point:** Average performance (moderate percentage of issues).
- **0 points:** Needs improvement (high percentage of issues).

The facility scoring system evaluates data completeness and timeliness using six critical metrics, assigning scores from 0 to 12. 
These metrics cover key areas such as timeliness of data entry, demographic information completeness, and accuracy of operational fields like funding source and lot numbers. 
Performance is categorized into four tiers—excellent, good, fair, and needing improvement—based on predefined thresholds.

## Usage
1. Execute the query in a database supporting SQL syntax.
2. Review results to identify high-performing facilities and those requiring intervention.

---

**Note:** Ensure that the database environment and schema names are correctly configured before running the query.
```
