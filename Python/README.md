# Identifying and quantifying exclusion of each characteristic (clinical speciality, medication use, age etc.) for RCTs in each drug category

The objective of the following code is to identify if an RCT excludes individuals with a specific characteristic, and to then calculate the overall exclusion count of each characteristic for all the RCTs in each drug category. 

Python version 3.8.10

## Input file

The input file is a csv file generated from our MySql database of RCTs, with the following template:

| id | name | type | clinical_study | condition_ | allocation | brief_title | phase | study_type | study_pop | criteria | gender | minimum_age | maximum_age | healthy_volunteers |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

Each characteristic has been manually added as an individual column prior to processing in Python. Example:

| id | name | type | clinical_study | condition_ | allocation | brief_title | phase | study_type | study_pop | criteria | gender | minimum_age | maximum_age | healthy_volunteers | medication_use | cardiovascular clinical speciality | ... |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |


## Importing packages 

```python
import pandas as pd 
import re
import os 
```

## Removing duplicates and unnecessary columns and rows

```python
for file in os.listdir(working_directory):
    print(file)
    df = pd.read_csv(working_directory + "/"+ file)
    drug_cat_name = file.replace(".csv","")

    df = df.drop_duplicates(subset=["clinical_study"], keep = "first")
    df = df.drop(["allocation"], axis = 1)
    df = df[~df.criteria.str.contains("<ct_uploader")]

    clin_trial_number = str(len(df))
    filename = "qc/" + drug_cat_name + "_" + clin_trial_number + ".csv"
    print(filename)
    df.to_csv(filename, index=False)
```

## Inspection of each category of RCTs to ensure that intervention for the RCTs only contain drugs for the respective category

```python 
# example check 
CVD_drug_list = drug_master_list["CVD_category"].tolist()
df[~df["name"].str.contains("|".join(CVD_drug_list), case = False)]
# returns any instance where the intervention is not in the drug category
```

## Standardising age format 

```python 
age_list = []
for file in os.listdir(qc):
    print(file)
    df = pd.read_csv(qc + "/"+ file)
    age_list = df["minimum_age"].tolist() + df["maximum_age"].tolist()

age_list = list(set(age_list))
age_list = [str(x) for x in age_list]
age_list = [x for x in age_list if x != "nan"]
print(age_list)

age_dict = dict.fromkeys(age_list)

for k, v in age_dict.items():
    if re.search("Year", k, flags = re.IGNORECASE):
        v = k.replace(" Years","").replace(" Year", "")
        age_dict[k] = int(v)
    if re.search("Month", k, flags = re.IGNORECASE):
        v = k.replace(" Months","").replace(" Month", "")
        v = int(v)/12
        age_dict[k] = v
    if re.search("Week", k, flags = re.IGNORECASE):
        v = k.replace(" Weeks","").replace(" Week", "")
        v = int(v)/52
        age_dict[k] = v
    if re.search("Day",k , flags = re.IGNORECASE):
        v = k.replace(" Days","").replace(" Day","")
        v=int(v)/365
        age_dict[k] = v
    if re.search("Hour", k, flags = re.IGNORECASE):
        v = 0 
        age_dict[k] = v  

for file in os.listdir(qc):
    print(file)
    df = pd.read_csv(qc + "/"+ file)
    
    df["minimum_age_QCed"] = df["minimum_age"].map(age_dict)
    df["maximum_age_QCed"] = df["maximum_age"].map(age_dict)
    df["minimum_age"] = df["minimum_age_QCed"].values
    df["maximum_age"] = df["maximum_age_QCed"].values
    df = df.drop(["minimum_age_QCed", "maximum_age_QCed"], axis = 1)

    df.to_csv(qc + "/"+ file, index=False)
```
