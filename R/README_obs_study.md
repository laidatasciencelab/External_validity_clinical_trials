# Cohort creation, identifying and quantifying prevalence of each characteristic (clinical speciality, medication use, age etc.) for individuals in observational study

The objective of the following code is to 1) create the observational study cohort and 2) to calculate the overall prevalence count of each characteristic for all individuals in the observational study cohort and 3) to run Cox regression analysis on propensity matched groups.

R version 3.6.2

## Input files
 1) CPRD cohort file

Master cohort file of patient IDs with patient variables (examples given) with the following template:

| patid | dob | entry date | V3 | V4 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

 2) CPRD diagnoses file with record of diagnoses for each patid

Diagnoses file of patient IDs with diagnoses and dates with the following template:

| patid | diagnosis 1 | date 1 | diagnosis 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

 3) CPRD prescription file with record of prescription for each patid 

 Prescription file where each prescription phenotype is a list of patient IDs with diagnoses and dates with the following template:

| patid | prescription 1 | date 1 | prescription 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

4) Clinical speciality binary file 

Clincal speciality file with the following template: 

| patid | clinical speciality 1 | date 1 | clinical speciality 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

5) Prescription binary file 

Prescription file with the following template: 

| patid | prescription phenotype 1 | date 1 | prescription phenotype 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Importing packages 

## Creating cohorts (incl. index age, no. of cs, no. of pres)

## Calculating prevalence numbers 

## Calculating mm and pp numbers 

## Propensity score matching 

## Cox regression analysis 

## Output files and tables 



