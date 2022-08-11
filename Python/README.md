# Identifying and quantifying exclusion of each characteristic (clinical speciality, medication use, age etc.) for RCTs in each drug category

The objective of the following code is to identify if an RCT excludes individuals with a specific characteristic, and to then calculate the overall exclusion count of each characteristic for all the RCTs in each drug category. 

Python version 3.8.10

## Input file

The input file is a csv file generated from our MySql database of RCTs, with the following template:

| id | name | type | clinical_study | condition_ | brief_title | phase | study_type | study_pop | criteria | gender | minimum_age | maximum_age | healthy_volunteers |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| id | name | type | clinical_study | condition_ | brief_title | phase | study_type | study_pop | criteria | gender | minimum_age | maximum_age | healthy_volunteers |

Each characteristic has been manually added as an individual column prior to processing in Python. Example:

| id | name | type | clinical_study | condition_ | brief_title | phase | study_type | study_pop | criteria | gender | minimum_age | maximum_age | healthy_volunteers | medication_use | cardiovascular clinical speciality |
