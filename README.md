# San Francisco Department of Public Health Inspection Data for 2013 - 2016

October 7, 2023

By:  Annie V Lam - Kura Labs

## Dataset

[San Francisco Department of Public Health Inspection Data for 2013 - 2016](https://c4databucket.s3.amazonaws.com/sanFranciscoRestaurantScores.zip)

There are three files uploaded to AWS S3 bucket:  businesses.csv, inspections.csv, and violations.csv.  The three sets of data were then imported to AWS Redshift inspections Dataabase the three files were uploaded as tables called businesses, inspections, and violations respectively.

## Cleaning the Inspection Dataset

**Add a new column field inspection_date to be formatted later**

-  ALTER TABLE inspections 
-  ADD COLUMN inspection_date DATE;

**Format inspection_date field to a date format**
-  UPDATE inspections
-  SET inspection_date = TO_DATE(date, 'YYYYMMDD');


UPDATE inspections SET inspection_date = TO_DATE(date, 'YYYYMMDD');


**How many inspection types are there?**

-  SELECT DISTINCT(type)
-  FROM inspections;

**Result:**

Complaint Reinspection/Followup	 score all NULL, New Construction, New Ownership, Foodborne Illness Investigation, Special Event, Multi-agency Investigation, Routine - Scheduled, Routine - Unscheduled, Reinspection/Followup, Non-inspection site visit, Complaint, Structural Inspection, Administrative or Document Review

**How many of the inspection types have scores?**

-  SELECT DISTINCT(type)
-  FROM inspections
-  WHERE score is not NULL;

**Result:**

New Ownership, Routine - Unscheduled, Reinspection/Followup

**Explore each of the three data types where the score is not NULL:**

-  SELECT *
-  FROM inspections
-  WHERE type = 'New Ownership' and score is not NULL
-  LIMIT 100;

**New Ownership Result:**

For New Ownership, only business id 87440 has a score for Aug 1, 2016.  Most likely this is an error where there should be no score or it was miscategorized as "New Ownership".  We would need to send this info to the business unit to verify.  

-  SELECT *
-  FROM inspections
-  WHERE type = 'Reinspection/Followup' and score is not NULL
-  LIMIT 100;

**Reinspection/Followup Result:**

For Reinspection/Followup, only business id 597 has a score for March 10, 2016.  Most likely this is an error where it was miss categorized as "New Ownership".  We would need to send this info to the business unit to verify. 

-  SELECT *
-  FROM inspections
-  WHERE type = 'Routine - Unscheduled' and score is not NULL
-  LIMIT 100;

**Routine - Unscheduled Result:**
All of the scores are from Routine - Unscheduled

## Exploring the Inspection Dataset for duplicate scores

**Create a key as save the query to a view**

-  CREATE VIEW inspection_view_0 AS
-  SELECT date+'-'+business_id as inspection_id,*
-  FROM inspections
-  WHERE score is not NULL AND type = 'Routine - Unscheduled';

**Find the duplicates and save it to a view**

-  CREATE VIEW duplicate_inspections AS
-  SELECT inspection_id, count(inspection_id)
-  FROM inspection_view_0
-  GROUP BY inspection_id
-  HAVING count(inspection_id) > 1;

**Explore the duplicate transactions**

-  SELECT i.* 
-  FROM duplicate_inspections d
-  LEFT JOIN inspection_view_0 i
-  ON d.inspection_id = i.inspection_id

**Result:**

There is a duplicate for business_id 64859 on September 24, 2015.  It received two scores:  93 and 96.  Per the business unit, the correct score is 96.

### Creating a Clean Inspection dataset

-  CREATE VIEW inspection_score_view AS
-  SELECT date+'-'+business_id as inspection_id, business_id , type as Type, inspection_date, score
-  FROM inspections
-  WHERE type = 'Routine - Unscheduled' AND score is not NULL AND date+business_id+CONVERT(varchar(10),score) <> '201509246485991';



## Cleaning the Violation Dataset

**ADD KEY to violations**

-  CREATE VIEW business_violations AS
-  SELECT SUBSTRING(date, 1, 4)+SUBSTRING(date, 6, 2)+SUBSTRING(date, 9, 2)+'-'+business_id+'-'+violationtypeid AS violation_id, *
-  FROM violations;

**VERIFY THERE ARE NO DUPLICATES in Violations**

-  SELECT violation_id, COUNT(violation_id) AS count_of_violation_id
-  FROM business_violations
-  GROUP BY violation_id
-  HAVING COUNT(violation_id) > 1;
  
**CREATE A VIOLATION_VIEW WITH NO DUPLICATES**

-  CREATE VIEW violation_view AS
-  WITH violations_tbl AS
-  (SELECT *, ROW_NUMBER()OVER(PARTITION BY violation_id ORDER BY violation_id) AS rownumber
-  FROM business_violations)
-  SELECT * FROM violations_tbl WHERE rownumber ='1' ORDER BY violation_id
-  

**CREATE VIEW for Duplicate Businesses**

-  CREATE VIEW duplicate_businesses AS
-  WITH businesses_tbl AS
-  (SELECT *, ROW_NUMBER()OVER(PARTITION BY business_id ORDER BY business_id) AS rownumber
-  FROM businesses)
-  SELECT * FROM businesses_tbl WHERE rownumber >'1' ORDER BY business_id




