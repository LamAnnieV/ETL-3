# San Francisco Department of Public Health Inspection Data for 2013 - 2016

October 7, 2023

By:  Annie V Lam - Kura Labs

## Dataset

[San Francisco Department of Public Health Inspection Data for 2013 - 2016](https://c4databucket.s3.amazonaws.com/sanFranciscoRestaurantScores.zip)

## Exploring the Data

There are three files uploaded to AWS S3 bucket:  businesses.csv, inspections.csv, and violations.csv.  The three sets of data were then imported to AWS Redshift inspections Dataabase the three files were uploaded as tables called businesses, inspections, and violations respectively.

**How many inspections types are there?**

SELECT DISTINCT(type)

FROM inspections;

**Result:**

Complaint Reinspection/Followup	 score all NULL, New Construction, New Ownership, Foodborne Illness Investigation, Special Event, Multi-agency Investigation, Routine - Scheduled, Routine - Unscheduled, Reinspection/Followup, Non-inspection site visit, Complaint, Structural Inspection, Administrative or Document Review

**How many of the inspection types have scores?**

SELECT DISTINCT(type)

FROM inspections

WHERE score is not NULL;

**Result:**

New Ownership, Routine - Unscheduled, Reinspection/Followup

**Explore each of the three data types where the score is not NULL:**

SELECT *

FROM inspections

WHERE type = 'New Ownership' and score is not NULL

LIMIT 100;

**Result: ** 

For New Ownership, only business id 87440 has a score for Aug 1, 2016.  Most likely this is an error where there should be no score or it was miscategorized as "New Ownership".  We would need to send this info to the business unit to verify.  

SELECT *

FROM inspections

WHERE type = 'Reinspection/Followup' and score is not NULL

LIMIT 100;

**Result: ** 

For Reinspection/Followup, only business id 597 has a score for March 10, 2016.  Most likely this is an error where it was miss categorized as "New Ownership".  We would need to send this info to the business unit to verify. 



SELECT *
FROM inspections
WHERE type = 'Routine - Unscheduled' and score is not NULL
LIMIT 100;


### SQL Queries to explore the data


