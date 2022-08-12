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

print(age_dict)

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

## Clinical speciality mapping file 

The mapping file is a csv file containing mapped condition phenotypes to a clinical speciality with the following template:



The input file is a csv file generated from our MySql database of RCTs, with the following template:

| condition | clinical_speciality_abbreviation | clinical_speciality |
| ------------- | ------------- | ------------- | 
|  |  |  |

## Clinical speciality mapping dictionary 

```python
clin_spec_mapping = pd.read_csv("clin_spec_mapping.csv", encoding = "latin-1")
speciality_list = clin_spec_mapping["clinical_speciality"].tolist()
speciality_dict = {k:[] for k in list(set(speciality_list))}

for k, v in speciality_dict.items():
    for row, index in enumerate(clin_spec_mapping.itertuples()):
        if str(k) == str(clin_spec_mapping.iloc[row,2]):
            v.append(str(clin_spec_mapping.iloc[row,0]))

print(speciality_dict)

# adding further medical terms to clinical specialities (example)
clin_spec_A = ["term_1", "term_2", "term_3"]
clin_spec_B = ["term_4", "term_5", "term_6"]
clin_spec_C = ["term_7", "term_8", "term_9"]

manual_value = [clin_spec_A, clin_spec_B, clin_spec_C]
manual_keys = ["clin_spec_A", "clin_spec_B", "clin_spec_C"]

alt_dict = {manual_keys[i]:manual_value[i] for i in range(len(manual_keys))}
speciality_dict = {key: specialty_dict[key] + alt_dict[key] if key in alt_dict.keys() else value for key, value in specialty_dict.items()}
```