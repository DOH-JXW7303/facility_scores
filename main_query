-- This query calculates a comprehensive facility score based on data quality and timeliness for vaccination records.
-- It evaluates six critical metrics and assigns a score ranging from 0 to 12 for each facility. 
-- High scores indicate excellent data management practices, while low scores highlight areas for improvement.

-- Step 1: Filter Vaccination Data for the Year 2024
WITH FilteredVaccinationData AS (
    SELECT * 
    FROM raw_waiis.vaccination_master
    WHERE YEAR(VACC_DATE) = 2024  -- Include only records where the vaccination date is in 2024
)

-- Main Query: Aggregate Metrics and Scores by Facility
SELECT 
    FM.ASIIS_FAC_ID AS Facility_ID,         -- Facility ID from the facility_master table
    FM.Name AS Facility_Name,               -- Facility name from the facility_master table

    -- Scores Column: Calculates the cumulative score for each facility
    (CASE 
        WHEN T.Beyond_One_Day_Percent <= 5 THEN 2  -- Timeliness score: 2 points for ≤ 5%
        WHEN T.Beyond_One_Day_Percent > 5 AND T.Beyond_One_Day_Percent <= 10.17 THEN 1  -- 1 point for > 5% and ≤ 10.17%
        ELSE 0  -- 0 points for > 10.17%
     END +
     CASE 
        WHEN V.Missing_VFC_Eligible_Percent <= 1 THEN 2  -- Missing VFC eligibility score: 2 points for ≤ 1%
        WHEN V.Missing_VFC_Eligible_Percent > 1 AND V.Missing_VFC_Eligible_Percent <= 2.87 THEN 1  -- 1 point for > 1% and ≤ 2.87%
        ELSE 0  -- 0 points for > 2.87%
     END +
     CASE 
        WHEN F.Missing_Funding_Source_Percent <= 1 THEN 2  -- Missing funding source score: 2 points for ≤ 1%
        WHEN F.Missing_Funding_Source_Percent > 1 AND F.Missing_Funding_Source_Percent <= 8.91 THEN 1  -- 1 point for > 1% and ≤ 8.91%
        ELSE 0  -- 0 points for > 8.91%
     END +
     CASE 
        WHEN L.Missing_Lot_Number_Percent <= 1 THEN 2  -- Missing lot number score: 2 points for ≤ 1%
        WHEN L.Missing_Lot_Number_Percent > 1 AND L.Missing_Lot_Number_Percent <= 18.74 THEN 1  -- 1 point for > 1% and ≤ 18.74%
        ELSE 0  -- 0 points for > 18.74%
     END +
     CASE 
        WHEN R.Missing_Race_Percent <= 1 THEN 2  -- Missing race score: 2 points for ≤ 1%
        WHEN R.Missing_Race_Percent > 1 AND R.Missing_Race_Percent <= 3.68 THEN 1  -- 1 point for > 1% and ≤ 3.68%
        ELSE 0  -- 0 points for > 3.68%
     END +
     CASE 
        WHEN E.Missing_Ethnicity_Percent <= 1 THEN 2  -- Missing ethnicity score: 2 points for ≤ 1%
        WHEN E.Missing_Ethnicity_Percent > 1 AND E.Missing_Ethnicity_Percent <= 6.14 THEN 1  -- 1 point for > 1% and ≤ 6.14%
        ELSE 0  -- 0 points for > 6.14%
     END
    ) AS Scores,  -- Total score for the facility (sum of all metric scores)

    -- Total Records (used across all metrics)
    COALESCE(T.Total_Records, 0) AS Total_Records,  -- Total vaccination records for the facility

    -- Timeliness Metrics
    T.Beyond_One_Day_Percent AS Beyond_One_Day,  -- Percentage of records entered more than one day after vaccination

    -- Missing Data Metrics
    V.Missing_VFC_Eligible_Percent AS Missing_VFC_Eligible,  -- Percentage of records missing VFC eligibility
    F.Missing_Funding_Source_Percent AS Missing_Funding_Source,  -- Percentage of records missing funding source
    L.Missing_Lot_Number_Percent AS Missing_Lot_Number,  -- Percentage of records missing or with invalid lot numbers
    R.Missing_Race_Percent AS Missing_Race,  -- Percentage of records missing race information
    E.Missing_Ethnicity_Percent AS Missing_Ethnicity  -- Percentage of records missing ethnicity information

FROM raw_waiis.facility_master AS FM  -- Facility master table

-- Subquery for Timeliness Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        COUNT(*) AS Total_Records,  -- Total vaccination records for the facility
        ROUND(100.0 * COUNT(CASE WHEN DATEDIFF(DAY, VM.VACC_DATE, VM.INSERT_STAMP) > 1 THEN 1 ELSE NULL END) / COUNT(*), 2) AS Beyond_One_Day_Percent
        -- Calculate the percentage of records entered more than one day after the vaccination date
    FROM FilteredVaccinationData AS VM
    GROUP BY VM.ASIIS_FAC_ID
) AS T
ON FM.ASIIS_FAC_ID = T.ASIIS_FAC_ID

-- Subquery for Missing VFC Eligible Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        ROUND(100.0 * COUNT(CASE WHEN VM.VFC_ELIGIBLE IS NULL THEN 1 ELSE NULL END) / COUNT(*), 2) AS Missing_VFC_Eligible_Percent
        -- Calculate the percentage of records with missing VFC eligibility information
    FROM FilteredVaccinationData AS VM
    GROUP BY VM.ASIIS_FAC_ID
) AS V
ON FM.ASIIS_FAC_ID = V.ASIIS_FAC_ID

-- Subquery for Missing Funding Source Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        ROUND(100.0 * COUNT(CASE WHEN VM.FUNDING_SOURCE_ID IS NULL THEN 1 ELSE NULL END) / COUNT(*), 2) AS Missing_Funding_Source_Percent
        -- Calculate the percentage of records with missing funding source information
    FROM FilteredVaccinationData AS VM
    GROUP BY VM.ASIIS_FAC_ID
) AS F
ON FM.ASIIS_FAC_ID = F.ASIIS_FAC_ID

-- Subquery for Missing Lot Number Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        ROUND(100.0 * COUNT(CASE WHEN VM.LOT_NUM IS NULL OR NOT EXISTS (
            SELECT 1 
            FROM raw_waiis.lot_number AS LN 
            WHERE VM.LOT_NUM = LN.LOT_NUMBER
        ) THEN 1 ELSE NULL END) / NULLIF(COUNT(*), 0), 2) AS Missing_Lot_Number_Percent
        -- Calculate the percentage of records with missing or invalid lot numbers
    FROM FilteredVaccinationData AS VM
    GROUP BY VM.ASIIS_FAC_ID
) AS L
ON FM.ASIIS_FAC_ID = L.ASIIS_FAC_ID

-- Subquery for Missing Race Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        ROUND(100.0 * SUM(CASE 
            WHEN NOT EXISTS (
                SELECT 1 
                FROM raw_waiis.PATIENT_RACE_RESERVE PRR 
                WHERE PRR.ASIIS_PAT_ID_PTR = PM.ASIIS_PAT_ID 
                  AND PRR.PAT_RACE <> '9'
            ) THEN 1 ELSE 0 
        END) / COUNT(*), 2) AS Missing_Race_Percent
        -- Calculate the percentage of records with missing race information
    FROM FilteredVaccinationData AS VM
    JOIN raw_waiis.PATIENT_MASTER PM
      ON VM.ASIIS_PAT_ID_PTR = PM.ASIIS_PAT_ID
    GROUP BY VM.ASIIS_FAC_ID
) AS R
ON FM.ASIIS_FAC_ID = R.ASIIS_FAC_ID

-- Subquery for Missing Ethnicity Metrics
LEFT JOIN (
    SELECT 
        VM.ASIIS_FAC_ID,
        ROUND(100.0 * SUM(CASE 
            WHEN NOT EXISTS (
                SELECT 1 
                FROM raw_waiis.PATIENT_RESERVE PR 
                WHERE PR.ASIIS_PAT_ID_PTR = PM.ASIIS_PAT_ID 
                  AND PR.PAT_ETHNICITY_CODE IN (1, 2)
            ) THEN 1 ELSE 0 
        END) / COUNT(*), 2) AS Missing_Ethnicity_Percent
        -- Calculate the percentage of records with missing ethnicity information
    FROM FilteredVaccinationData AS VM
    JOIN raw_waiis.PATIENT_MASTER PM
      ON VM.ASIIS_PAT_ID_PTR = PM.ASIIS_PAT_ID
    GROUP BY VM.ASIIS_FAC_ID
) AS E
ON FM.ASIIS_FAC_ID = E.ASIIS_FAC_ID

-- Filtering and Final Ordering
WHERE FM.ASIIS_FAC_ID IS NOT NULL  -- Ensure facility IDs are valid
  AND NOT (
      T.Beyond_One_Day_Percent IS NULL
      AND V.Missing_VFC_Eligible_Percent IS NULL
      AND F.Missing_Funding_Source_Percent IS NULL
      AND L.Missing_Lot_Number_Percent IS NULL
      AND R.Missing_Race_Percent IS NULL
      AND E.Missing_Ethnicity_Percent IS NULL
  )
ORDER BY 
    Scores DESC,  -- Facilities with higher scores appear first
    T.Total_Records DESC,  -- Break ties with total record count
    T.Beyond_One_Day_Percent DESC,
    V.Missing_VFC_Eligible_Percent DESC,
    F.Missing_Funding_Source_Percent DESC,
    L.Missing_Lot_Number_Percent DESC,
    R.Missing_Race_Percent DESC,
    E.Missing_Ethnicity_Percent DESC;
