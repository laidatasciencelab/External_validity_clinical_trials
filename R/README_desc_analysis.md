# Cohort creation, identifying and quantifying prevalence of each characteristic (clinical speciality, medication use, age etc.) for individuals in each drug category cohort

The objective of the following code is to 1) create the different drug category cohorts and 2) to calculate the overall prevalence count of each characteristic for all individuals in the each drug category cohort.

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

4) Clinical speciality to condition mapping file

Mapping file where each condition is mapped to a clinical speciality with the following template: 

| clinical speciality | condition | 
| ------------- | ------------- |
| CVD | CVD condition 1 |
| CVD | CVD condition 2 |
| ... | ... |

5) Drug cohort category to prescription mapping file 

Mapping file where each prescription is mapped to a drug cohort category with the following template: 

| drug cohort category | prescription | 
| ------------- | ------------- |
| CVD | CVD prescription phenotype 1 |
| CVD | CVD prescription phenotype 2 |
| ... | ... |

## Importing packages 

```R
library(tidyverse)
library(data.table)
library(tableone)
library(lubridate)
library(dplyr)
library(purrr)
```

## Creating clinical speciality binary file 

```R
# Load in cprd diagnoses file 
cprd_diagnoses = readRDS("cprd_diagnoses.rds")

# df with patids from diagnoses file 
clin_spec = cprd_diagnoses[,1 , drop = FALSE]

# example with CVD clinical speciality 
# extract all columns with CVD conditions and dates 
CARDtemp = cprd_diagnoses = [, c(
    "eventdate.cal_Atrial_fibrillation_cprd_1",
    "eventdate.cal_Atrioventricular_block_complete_cprd_1",
    "eventdate.cal_Atrioventricular_block_degree_cprd_1",
    "eventdate.cal_Atrioventricular_block_second_degree_cprd_1",
    "eventdate.cal_Bifascicular_block_1",
    "eventdate.cal_Congenital_malformations_cardiac_septa_1",
    "eventdate.cal_Coronary_heart_disease_nos",
    "eventdate.cal_Dilated_cardiomyopathy_1",
    "eventdate.cal_Heart_failure_1",
    "eventdate.cal_Hypertension_1",
    "eventdate.cal_Hypertrophic_Cardiomyopathy_1",
    "eventdate.cal_Left_bundle_branch_block_1",
    "eventdate.cal_Multiple_valve_dz_1",
    "eventdate.cal_Myocardial_infarction_1",
    "eventdate.cal_Nonrheumatic_aortic_valve_disorders_1",
    "eventdate.cal_Nonrheumatic_mitral_valve_disorders_1",
    "eventdate.cal_Other_Cardiomyopathy_1",
    "eventdate.cal_Patent_ductus_arteriosus_1",
    "eventdate.cal_Pericardial_effusion_noninflammatory_1",
    "eventdate.cal_Primary_pulmonary_hypertension_1",
    "eventdate.cal_Rheumatic_fever_1",
    "eventdate.cal_Rheumatic_valve_dz_1",
    "eventdate.cal_Right_bundle_branch_block_1",
    "eventdate.cal_Sick_sinus_syndrome_cprd_1",
    "eventdate.cal_Stable_angina_cprd_1",
    "eventdate.cal_Supraventricular_tachycardia_cprd_1",
    "eventdate.cal_Trifascicular_block_1",
    "eventdate.cal_Unstable_Angina_cprd_1",
    "eventdate.cal_Ventricular_tachycardia_cprd_1",
    "cal_Atrial_fibrillation_cprd_1",
    "cal_Atrioventricular_block_complete_cprd_1",
    "cal_Atrioventricular_block_degree_cprd_1",
    "cal_Atrioventricular_block_second_degree_cprd_1",
    "cal_Bifascicular_block_1",
    "cal_Congenital_malformations_cardiac_septa_1",
    "cal_Coronary_heart_disease_nos",
    "cal_Dilated_cardiomyopathy_1",
    "cal_Heart_failure_1",
    "cal_Hypertension_1",
    "cal_Hypertrophic_Cardiomyopathy_1",
    "cal_Left_bundle_branch_block_1",
    "cal_Multiple_valve_dz_1",
    "cal_Myocardial_infarction_1",
    "cal_Nonrheumatic_aortic_valve_disorders_1",
    "cal_Nonrheumatic_mitral_valve_disorders_1",
    "cal_Other_Cardiomyopathy_1",
    "cal_Patent_ductus_arteriosus_1",
    "cal_Pericardial_effusion_noninflammatory_1",
    "cal_Primary_pulmonary_hypertension_1",
    "cal_Rheumatic_fever_1",
    "cal_Rheumatic_valve_dz_1",
    "cal_Right_bundle_branch_block_1",
    "cal_Sick_sinus_syndrome_cprd_1",
    "cal_Stable_angina_cprd_1",
    "cal_Supraventricular_tachycardia_cprd_1",
    "cal_Trifascicular_block_1",
    "cal_Unstable_Angina_cprd_1",
    "cal_Ventricular_tachycardia_cprd_1",
    "patid")]

# identifying any patid who has >0 of the above conditions to be positive ("1") for the CVD clinical speciality 
CARDtemp$CARDclin = ifelse(CARDtemp$cal_Atrial_fibrillation_cprd_1== "1"|
                            CARDtemp$cal_Atrioventricular_block_complete_cprd_1== "1"|
                            CARDtemp$cal_Atrioventricular_block_degree_cprd_1== "1"|
                            CARDtemp$cal_Atrioventricular_block_second_degree_cprd_1== "1"|
                            CARDtemp$cal_Bifascicular_block_1== "1"|
                            CARDtemp$cal_Congenital_malformations_cardiac_septa_1== "1"|
                            CARDtemp$cal_Coronary_heart_disease_nos== "1"|
                            CARDtemp$cal_Dilated_cardiomyopathy_1== "1"|
                            CARDtemp$cal_Heart_failure_1== "1"|
                            CARDtemp$cal_Hypertension_1== "1"|
                            CARDtemp$cal_Hypertrophic_Cardiomyopathy_1== "1"|
                            CARDtemp$cal_Left_bundle_branch_block_1== "1"|
                            CARDtemp$cal_Multiple_valve_dz_1== "1"|
                            CARDtemp$cal_Myocardial_infarction_1== "1"|
                            CARDtemp$cal_Nonrheumatic_aortic_valve_disorders_1== "1"|
                            CARDtemp$cal_Nonrheumatic_mitral_valve_disorders_1== "1"|
                            CARDtemp$cal_Other_Cardiomyopathy_1== "1"|
                            CARDtemp$cal_Patent_ductus_arteriosus_1== "1"|
                            CARDtemp$cal_Pericardial_effusion_noninflammatory_1== "1"|
                            CARDtemp$cal_Primary_pulmonary_hypertension_1== "1"|
                            CARDtemp$cal_Rheumatic_fever_1== "1"|
                            CARDtemp$cal_Rheumatic_valve_dz_1== "1"|
                            CARDtemp$cal_Right_bundle_branch_block_1== "1"|
                            CARDtemp$cal_Sick_sinus_syndrome_cprd_1== "1"|
                            CARDtemp$cal_Stable_angina_cprd_1== "1"|
                            CARDtemp$cal_Supraventricular_tachycardia_cprd_1== "1"|
                            CARDtemp$cal_Trifascicular_block_1== "1"|
                            CARDtemp$cal_Unstable_Angina_cprd_1== "1"|
                            CARDtemp$cal_Ventricular_tachycardia_cprd_1== "1", 1, 0)

# minimising workflow 
CARDtemp = CARDtemp[!(CARDtemp$CARDclin==0)]

# identifying earliest diagnosis date for CVD clinical speciality for each patid 
CARDdate = CARDtemp[,c("eventdate.cal_Atrial_fibrillation_cprd_1",
                       "eventdate.cal_Atrioventricular_block_complete_cprd_1",
                       "eventdate.cal_Atrioventricular_block_degree_cprd_1",
                       "eventdate.cal_Atrioventricular_block_second_degree_cprd_1",
                       "eventdate.cal_Bifascicular_block_1",
                       "eventdate.cal_Congenital_malformations_cardiac_septa_1",
                       "eventdate.cal_Coronary_heart_disease_nos",
                       "eventdate.cal_Dilated_cardiomyopathy_1",
                       "eventdate.cal_Heart_failure_1",
                       "eventdate.cal_Hypertension_1",
                       "eventdate.cal_Hypertrophic_Cardiomyopathy_1",
                       "eventdate.cal_Left_bundle_branch_block_1",
                       "eventdate.cal_Multiple_valve_dz_1",
                       "eventdate.cal_Myocardial_infarction_1",
                       "eventdate.cal_Nonrheumatic_aortic_valve_disorders_1",
                       "eventdate.cal_Nonrheumatic_mitral_valve_disorders_1",
                       "eventdate.cal_Other_Cardiomyopathy_1",
                       "eventdate.cal_Patent_ductus_arteriosus_1",
                       "eventdate.cal_Pericardial_effusion_noninflammatory_1",
                       "eventdate.cal_Primary_pulmonary_hypertension_1",
                       "eventdate.cal_Rheumatic_fever_1",
                       "eventdate.cal_Rheumatic_valve_dz_1",
                       "eventdate.cal_Right_bundle_branch_block_1",
                       "eventdate.cal_Sick_sinus_syndrome_cprd_1",
                       "eventdate.cal_Stable_angina_cprd_1",
                       "eventdate.cal_Supraventricular_tachycardia_cprd_1",
                       "eventdate.cal_Trifascicular_block_1",
                       "eventdate.cal_Unstable_Angina_cprd_1",
                       "eventdate.cal_Ventricular_tachycardia_cprd_1")]

temp = do.call(pmin, c(CARDdate, list(na.rm = TRUE)))
CARDtemp$CARD_index_date <- temp
CARDtemp = CARDtemp[,c(ncol(CARDtemp)-2):ncol(CARDtemp)]

# merge back into main clinical speciality df 
clin_spec = merge(x = clin_spec, y = CARDtemp, by = "patid", all.x = TRUE)

# ... repeat for 27 remaining clinical specialities 
```
### Output

| patid | clinical speciality 1 | date 1 | clinical speciality 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Creating prescription binary file 

```R
# Load in cprd cohort file 
cohort_full = readRDS("cprd_cohort_file.rds")

# Load in cprd diagnoses file 
cprd_prescriptions = readRDS("cprd_prescriptions.rds")

# df with patids from cohort file 
pres_merged = cohort_full[,1 , drop = FALSE]

# example with generic prescription phenotype (i.e. phenotype 1)
# extracting the phenotype information 
GP1 = as.data.frame(cprd_prescriptions[1])

# formatting the dataframe and extracting earliest prescription date out of multiple prescriptions in the same phenotype 
rowname(GP1) = GP1$patid
GP1$temphold = NA 
c = 1:ncol(GP1)
GP1 = GP1[,c%%2==0]
tempp = do.call(pmin, c(GP1, list(na.rm = TRUE)))
GP1$prescription_index_date = tempp
GP1$patid = rownames(GP1)
GP1 <- GP1[tail(names(GP1),2)]
GP1 <- GP1[,c(2,1)]
names(GP1)[2] = "GP1_processed"

# merge back into main prescription df
pres_merged = merge(x = pres_merged, y = GP1, by = "patid", all.x = TRUE) 

# ... repeat for 132 remaining prescription phenotypes 
```
### Output

| patid | prescription phenotype 1 | date 1 | prescription phenotype 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Creating drug category cohorts (incl. index age, no. of cs, no. of pres)

## Calculating prevalence numbers 

## Calculating mm and pp numbers 

## Output files and tables 



