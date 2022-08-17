# Cohort creation, identifying and quantifying prevalence of each characteristic (clinical speciality, medication use, age etc.) for individuals in observational study

The objective of the following code is to 1) create the observational study cohort and 2) to calculate the overall prevalence count of each characteristic for all individuals in the observational study cohort and 3) to run Cox regression analysis on propensity matched groups.

R version 3.6.2

## Input files
 1) CPRD cohort file

Wrangled master cohort file of patient IDs with patient variables (examples given) with the following template:

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

```R
library(tidyverse)
library(data.table)
library(tableone)
library(lubridate)
library(dplyr)
library(purrr)
library(broom)
library(survminer)
library(survival)
library(survMisc)
library(MatchIt)
library(prodlim)
```

## Creating observational study cohort

```R
# load in prescription binary file 
pres_merged = readRDS("prescription_binary.rds")

# load in cprd diagnoses file 
cprd_diagnoses = readRDS("cprd_diagnoses.rds")

# date-only version of cprd diagnoses file 
c = 1:ncol(cprd_diagnoses)
diagnoses_dates = cprd_diagnoses[, c%%2==0]

# creating base cohort of individuals with an anti-dementia prescription and a prior dementia diagnosis 
dc = pres_merged[c("patid","Drugs_for_dementia")] %>% drop_na(Drugs_for_dementia)]
dm_diagnosis = cprd_diagnoses[c("patid", "eventdate.cal_Dementia_1")]
dcm = merge(x = dc, y = dm_diagnosis, by = "patid", all.x = TRUE)
dcm = dcm %>% drop_na(eventdate.cal_Dementia_1)
colnames(dcm)[2] = "AD_prescription_date"
colnames(dcm)[3] = "AD_diagnosis date"

# ensure that diagnosis date precedes index prescription date
dcm$difference = difftime(dcm$AD_prescription_date,dcm$AD_diagnosis_date, units = "days")
dcm = dcm %>% 
        mutate(difference = replace(difference, difference < 0, 0)) %>% 
        mutate(difference = replace(difference, difference > 0, 1)) %>%
        filter(dcm, difference==1)

# identifying all in the cohort with a concomitant CVD condition 
dcm = merge(x = dcm, y = diagnoses_dates, by = "patid", all.x = TRUE)
dcm_card = dcm[c(1:3,29:32,60,100,125,148,266,272,285)]

# construct index date for CVD diagnosis
dcm_temp = dcm_card[c(1:2,4:14)]
dcm_cardtemp = dcm_temp[c(3:13)]
temp = do.call(pmin, c(dcm_cardtemp, list(na.rm = TRUE)))
dcm_card$CARD_index_date = temp

# construct contraindication status 
# ensure that diagnosis date precedes index prescription date
dcm_card$carddiff = difftime(dcm_card$AD_prescription_date,dcm_card$CARD_index_date,units="days")
dcm_card$carddiff[dcm_card$carddiff < 0 | is.na(dcm_card$carddiff)] = 0 
dcm_card$carddiff[dcm_card$carddiff > 0] = 1
dcm_card = dcm_card[c(1:2,15:16)]

# identifying all in the cohort with other conditions 
dcm_indi = dcm[c(1:28,33:59,61:63,65:99,101:124,126:147,149:265,267:271,273:284,286:290)]

# construct index date for other diagnoses
dcm_inditemp = dcm_indi[c(5:278)]
tempv = do.call(pmin, c(dcm_inditemp, list(na.rm = TRUE)))
dcm_indi$indi_date = tempv

# construct other indication status 
dcm_indi = dcm_indi[c(1:2,279)]
dcm_indi$indidiff = difftime(dcm_indi$AD_prescription_date,dcm_indi$indi_date,units="days")
dcm_indi$indidiff[dcm_indi$indidiff < 0] <- 0 
dcm_indi$indidiff[dcm_indi$indidiff > 0] <- 1

# merge all information into single df 
dcmf = merge(x = dcm_card, y = dcm_indi, by = "patid", all.x = TRUE)

# qc to ensure individuals must have a contraindication or indication 
dcmf = dcmf[which(dcmf$carddiff=='1' | dcmf$indidiff=='1'), ]

# formatting
dcmf[, c(4,7)] <- sapply(dcmf[, c(4,7)], as.numeric)
colnames(dcmf)[2] <- 'AD_prescription_date'
colnames(dcmf)[4] <- 'Before_index_status_CARD'
colnames(dcmf)[7] <- 'Before_index_status_indi'

# re-organising groupings to contraindication and no contraindication 
dcmf$Before_index_status_indi <- ifelse(dcmf$Before_index_status_CARD=="1",0,1)
dcmf = dcmf[which(dcmf$Before_index_status_CARD == '1' | dcmf$Before_index_status_CARD == '0'), ]
```
## Calculating counts of concomitant clinical specialties and prescriptions per patid 



## Calculating prevalence numbers 

## Calculating mm and pp numbers 

## Propensity score matching 

## Cox regression analysis 

## Output files and tables 



