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

## QC for cohort main cohort file 
```R
# load in cohort file 
cohort_full = readRDS("cprd_cohort_file.rds")
cohort_full$baseline_date = ymd(cohort_full$baseline_date)
cohort_full$date_exit = ymd(cohort_full$date_exit)
cohort_full$dob = ymd(cohort_full$dob)

# QC sex
cohort_full = cohort_full[!cohort_full$sex==3]

# QC age conditions 
# condition: not lived/be living should not be longer than 119 years 
# condition: data start should not exceed data end
cohort_full$QC1 = difftime(cohort_full$date_exit, cohort_full$dob, unit = "weeks") 
cohort_full$QC1 = ifelse(cohort_full$QC1 < 6257.148, 0, 1)
cohort_full = cohort_full[!cohort_full$QC1==1]

cohort_full$QC2 = ifelse(!is.na(cohort_full$date_exit), difftime(cohort_full$date_exit, cohort_full$dob, units = "weeks"), difftime("31-12-2020", cohort_full$dob, units = "weeks"))
cohort_full$QC2 = ifelse(cohort_full$QC2 < 6257.148, 0, 1)
cohort_full = cohort_full[!cohort_full$QC2==1]

cohort_full$QC3 = difftime(cohort_full$date_exit, cohort_full$baseline_date, unit = "weeks") 
cohort_full$QC3 = felse(cohort_full$QC3 > 0, 0, 1)
cohort_full = cohort_full[!cohort_full$QC3==1]

cohort_full$QC1 = NULL
cohort_full$QC2 = NULL
cohort_full$QC3 = NULL

# final number 6,163,418
```

## Creating clinical speciality binary file 

```R
# load in cprd diagnoses file 
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
# load in cprd prescriptions file 
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
GP1 = GP1[tail(names(GP1),2)]
GP1 = GP1[,c(2,1)]
names(GP1)[2] = "GP1_processed"

# merge back into main prescription df
pres_merged = merge(x = pres_merged, y = GP1, by = "patid", all.x = TRUE) 

# ... repeat for 132 remaining prescription phenotypes 
```
### Output

| patid | prescription phenotype 1 | date 1 | prescription phenotype 2 | date 2 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Creating base drug category cohorts 

```R
# example with CVD drug category cohort 
# identifying dfs with CVD prescription phenotypes via mapping file 
# CVD system, GP115, GP51, GP11, GP34, GP86, GP105, GP134, GP18, GP26, GP132, GP21, GP92, GP97

# format as individual df
CVD115 = as.data.frame(cprd_prescriptions[115])
CVD51 = as.data.frame(cprd_prescriptions[51])
CVD11 = as.data.frame(cprd_prescriptions[11])
CVD34 = as.data.frame(cprd_prescriptions[34])
CVD86 = as.data.frame(cprd_prescriptions[86])
CVD105 = as.data.frame(cprd_prescriptions[105])
CVD134 = as.data.frame(cprd_prescriptions[134])
CVD18 = as.data.frame(cprd_prescriptions[18])
CVD26 = as.data.frame(cprd_prescriptions[26])
CVD132 = as.data.frame(cprd_prescriptions[132])
CVD21 = as.data.frame(cprd_prescriptions[21])
CVD92 = as.data.frame(cprd_prescriptions[92])
CVD97 = as.data.frame(cprd_prescriptions[97])

# transform into list for formatting 
list.cvd = list((CVD115,
                 CVD51,
                 CVD11,
                 CVD34,
                 CVD86,
                 CVD105,
                 CVD134,
                 CVD18,
                 CVD26,
                 CVD132,
                 CVD21,
                 CVD92,
                 CVD97))

# formatting the dataframe and extracting earliest prescription date out of multiple prescriptions in the same phenotype 
CVD_processed = lapply(
     list.cvd,
  function(df) {
    rownames(df)= df$patid
    df$temphold = NA
    c <- 1:ncol(df)
    df = df[,c%%2==0]
    min_date = do.call(pmin, c(df, list(na.rm = TRUE)))
    df$prescription_index_date = min_date
    df$patid = rownames(df)
    df <- df[tail(names(df),2)]
    return(df)
  }
)

# combine list of df into single cohort 
CVD_cohort = bind_rows(CVD_processed)

# formatting and QC 
CVD_cohort = CVD_cohort 
                %>% drop_na(prescription_index_date) 
                %>% group_by(patid)
                %>% filter(prescription_index_date == min(prescription_index_date))
CVD_cohort$drug_Cat = 2
CVD_cohort <- CVD_cohort[,c(2,4,3,1)]
CVD_cohort = CVD_cohort %>% filter(patid in cohort_full)

# merging with main cohort file 
CVD_merged = merge(x = CVD_cohort, y = cohort_full, by = "patid", all.x = TRUE)

# calculate age based on index prescription
CVD_merged$prescription_index_age = interval(start = CVD_merged$dob, end = CVD_merged$prescription_index_date)/duration(n=1, unit="years")

# QC age 
CVD_merged$qc = ifelse(CVD_merged$prescription_index_age <0, 1, 0)
CVD_merged = CVD_merged[!CVD_merged$qc==1]

# total 
nrow(CVD_merged) 
CVD_cohort = CVD_merged
# total 2,462,852 for CVD cohort

# ... repeat for 12 remaining drug category cohorts
```
### Output

| patid | dob | prescription index date | prescription index age | X1 | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Calculating counts of concomitant clinical specialties and prescriptions per patid per drug category cohort

```R
# example with CVD drug category cohort 
# calculating counts of concomitant prescriptions per patid 
df_cohort = list(CVD_cohort)
df_processed = lapply(
    df_cohort,
    function(df){
        merged_df = merge(df, pres_merged, by = "patid", all.x = TRUE)
        merged_df = merged_df[c(2:135)]
        foo = function(x){
        date = merged_df$prescription_index_date + 1
        merged_df$x <- difftime(date , x , units = "days")
        }
        merged_df = sapply(merged_df, foo)
        # ensures prescription does not precede index prescription and the concomitant prescription is given within a year of the index prescription
        merged_df[merged_df < 0 | is.na(merged_df) | merged_df > 365] <- 0
        merged_df[merged_df > 0] <- 1
        df$pres_quantity = rowSums(merged_df)
        return(df)
  }
)

CVD_cohort = as.data.frame(df_processed[1])

# calculating counts of concomitant clinical speciality per patid 
c = 1:ncol(clin_spec)
clin_spec = clin_spec[,c%%2==1, with = FALSE]

df_cohort = list(CVD_cohort)
df_processed = lapply(
  df_cohort,
  function(df){
    merged_df = merge(df, clin_spec, by = "patid", all.x = TRUE)
    merged_df = merged_df[c(2,4:31)]
    foo = function(x){
      date = merged_df$prescription_index_date + 1
      merged_df$x <- difftime(date, x, units = "days")
    }
    merged_df = sapply(merged_df, foo)
    # ensures diagnoses does not precede index prescription
    merged_df[merged_df < 0 | is.na(merged_df)] <- 0
    merged_df[merged_df > 0] <- 1
    df$clin_spec_quantity = rowSums(merged_df)
    return(df)
  }
)

CVD_cohort = as.data.frame(df_processed[1])

# removing a count of 2 from pres_quantity to account for 1x prescription index date and 1x actual index date in df

# removing a count of 1 from clin_spec_quantity to account for 1x prescription index date in df

df_cohort = list(CVD_cohort)
df_processed = lapply(
  df_cohort,
  function(df){
    df$pres_quantity = df$pres_quantity - 2
    df$clin_spec_quantity = df$clin_spec_quantity - 1
    return(df)
  }
)

CVD_cohort = as.data.frame(df_processed[1])

# ... repeat for 12 remaining drug category cohorts
```

### Output

| patid | dob | ... | pres_quantity | clin_spec_quantity | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Calculating counts by clinical speciality per drug category cohort for prevalence calculations

```R
# example with CVD drug category cohort 

df_cohort = list(CVD_cohort)
prev_count = lapply(
  df_cohort,
  function(df){
    merged_df = merge(df, clin_spec, by = "patid", all.x = TRUE)
    merged_df = merged_df[, c(2:30)]
    foo = function(x){
      date = merged_df$prescription_index_date + 1
      merged_df$x = difftime(date,x,units="days")
    }
    merged_df = sapply(merged_df, foo)
    # ensures diagnoses does not precede index prescription
    merged_df[merged_df < 0 | is.na(merged_df)] <- 0
    merged_df[merged_df > 0] <- 1
    output = colSums(merged_df)
    return(output)
  }
)

CVD_prev_count = as.data.frame(prev_count[1])
```

### Output

| drug category cohort | clin_spec 1 count | clin_spec 2 count | ... | ... | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
| CVD | ... | ... |  |  |  |

## Calculating summary counts of concomitant clinical specialties and prescriptions per drug category cohort binned by age groups

```R
```

## Calculating top 10 conditions and prescriptions for internal analysis of anomalies per drug category cohort binned by age groups 

```R
```


