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
# load in cohort file
cohort_full = readRDS("cohort_full.rds")

# load in prescription binary file 
pres_merged = readRDS("prescription_binary.rds")

# load in cprd diagnoses file 
cprd_diagnoses = readRDS("cprd_diagnoses.rds")

# load in clinical speciality binary file
clin_spec = readRDS("clin_spec_binary.rds")

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

```R
# calculating counts of concomitant clinical speciality per patid 
dcmfmm = merge(x = dcmf, y = clin_spec, by = "patid", all.x = TRUE)
dcmfmm = dcmfmm[c(2,7:62)]
c = 1:ncol(dcmfmm)
dcmfmmtemp = dcmfmm[,c%%2==1]
foo = function(x){
  date = dcmfmmtemp$AD_prescription_date + 1
  dcmfmmtemp$x <- difftime(date, x, units = "days")
}
dcmfmmtemp <- sapply(dcmfmmtemp, foo)
dcmfmmtemp = data.frame(dcmfmmtemp)

# ensure that diagnosis date precedes index prescription date
dcmfmmtemp[dcmfmmtemp < 0 | is.na(dcmfmmtemp)] <- 0
dcmfmmtemp[dcmfmmtemp > 0] <- 1
dcmfmmtemp$AD_prescription_date =NULL
dcmf$clinspec_total = rowSums(dcmfmmtemp)

# calculating counts of concomitant prescriptions per patid 
dcmfpp = merge(x = dcmf, y = pres_merged, by = "patid", all.x = TRUE)
dcmfpp = dcmfpp[c(2,8:57,59:140)]
foo = function(x){
  date = dcmfpp$AD_prescription_date+1
  dcmfpp$x <- difftime(date,x,units="days")
}
dcmfpp <- sapply(dcmfpp, foo)
dcmfpp = data.frame(dcmfpp)

# ensure that prescription date precedes index prescription date
dcmfpp[dcmfpp < 0 | is.na(dcmfpp) | dcmfpp > 365] <- 0
dcmfpp[dcmfpp > 0] <- 1
dcmfpp$AD_prescription_date =NULL
dcmf$pres_total = rowSums(dcmfpp)

# merge with cohort_full 
dcmf_ready = merge(x = dcmf,y = cohort_full,by="patid",all.x=TRUE)
dcmf_ready = dcmf_ready[c(1:8,10:15,17)]

# qc to ensure exit date does not precede analysis period 
dcmf_ready$qc = difftime(dcmf_ready$date_exit,dcmf_ready$AD_prescription_date,units="days")
dcmf_ready$qc <- ifelse(dcmf_ready$qc<0,1,0)
dcmf_ready = dcmf_ready[which(dcmf_ready$qc=='0'), ]
```

## Calculating prevalence numbers 

```R
dcmfr = dcmf_ready[c(1:2)]
dcmfr = merge(dcmfr, clin_spec, by = "patid", all.x = TRUE)
c = 1:ncol(dcmf)
dcmftemp = dcmfr[,c%%2==0]
foo = function(x){
  date = dcmftemp$AD_prescription_date + 1
  dcmftemp$x <- difftime(date,x,units="days")
}
dcmftemp <- sapply(dcmftemp, foo)
dcmftemp = data.frame(dcmftemp)

# ensure that diagnosis date precedes index prescription date 
dcmftemp[dcmftemp < 0 | is.na(dcmftemp)] <- 0
dcmftemp[dcmftemp > 0] <- 1
dcmftemp$AD_prescription_date =NULL

output = colSums(dcmftemp)
output$condition = colnames(dcmftemp)
output = data.frame(output)
```

## Pre-matching wrangling
```R
# age at prescription
dcmf_ready$age_index = interval(start = dcmf_ready$dob, end = dcmf_ready$AD_prescription_date)/duration(n = 1, unit = "years")

# survival years 
dcmf_ready$os_yrs = interval(start = dcmf_ready$AD_prescription_date, end = dcmf_ready$date_exit)/duration(n = 1, unit = "years")

# limit follow up to 1-10 years 
dcmf_ready$dead = ifelse(dcmf_ready$os_yrs > 10, 0, dcmf_ready$dead)
dcmf_ready$os_yrs = ifelse(dcmf_ready$os_yrs > 10, 10, dcmf_ready$os_yrs)
dcmf_ready$followup = difftime(dcmf_ready$date_exit,dcmf_ready$AD_prescription_date,units="days")
dcmf_ready$followup = as.numeric(dcmf_ready$followup)
dcmf_ready$followup = ifelse(dcmf_ready$followup < 365 & dcmf_ready$dead == "0", 0, 1)
dcmf_ready = dcmf_ready[which(dcmf_ready$followup == "1"), ]
dcmf_ready$followup = NULL

# median follow up time 
dcm.exposed = dcm.matched[which(dcm.matched$Before_index_status_CARD=="1"), ]
dcm.control = dcm.matched[which(dcm.matched$Before_index_status_CARD=="0"), ]
quantile(prodlim(Hist(time=os_yrs,event=dead)~1, 
                 data = dcm.exposed, reverse=TRUE))
quantile(prodlim(Hist(time=os_yrs,event=dead)~1, 
                 data = dcm.control, reverse=TRUE))
```

## Propensity score matching 
```R
# selecting necessary columns 
dcmfp <- dcmf_ready[c(1,4,6:10,14,17:18)]

# baseline characteristics table 
table1<- CreateTableOne(vars = c('clinspec_total','pres_total','prac_region','gender','age_index'),
                        data = dcmfp,
                        factorVars = c('gender','prac_region'),
                        strata = 'Before_index_status_CARD')

# propensity score matching 
set.seed(8989)
dcmfp$group <- as.logical(dcmfp$Before_index_status_CARD == "1")
match.it <-matchit(group ~ clinspec_total+pres_total+prac_region+gender+age_index,data=dcmfp,method="nearest",ratio=1,link="logit",caliper= c(.2),std.caliper=TRUE)

matched = summary(match.it)
matched
matched$sum.matched

# propensity score distribution 
plot(match.it, type='jitter', interactive=FALSE)

# baseline characteristics table after matching
table4<- CreateTableOne(vars = c('clinspec_total','pres_total','prac_region','gender','age_index'),
                        data = dcm.matched,
                        factorVars = c('gender','prac_region'),
                        strata = 'Before_index_status_CARD')

# baseline characteristics of cohort 
myvars = c("age_index", "prac_region","gender", "clinspec_total", "pres_total")
catvars = c("prac_region","gender","clinspec_total", "pres_total")
baseline_table = CreateTableOne(vars = myvars, data = dcm.matched, factorVars = catvars)
summary(baseline_table)
baseline_output = print(baseline_table)

# matched cohort 
dcm.matched = match.data(match.it)[1:ncol(dcmfp)]

# repeat for other conditions (e.g. single psychiatric conditions) to form matched cohorts from the base dementia cohort
```

## Cox regression analysis 
```R
# basic plot of KM graph 
km_fit_strata = survfit(Surv(os_yrs,dead)~Before_index_status_CARD, data=dcm.matched)
plot(km_fit_strata,xlabs="years", main="KM plot")

# survminer theme 
custom_theme <- function() {
  theme_survminer() %+replace%
    theme(
      plot.title = element_text(hjust=0.5, face="bold")
    )
}

# ggplot of KM graph 
fit = survfit(Surv(os_yrs,dead) ~ Before_index_status_CARD, data=dcm.matched)
summary(fit)

ggsurvplot(
  fit = survfit(Surv(os_yrs,dead) ~ Before_index_status_CARD, data=dcm.matched),
  conf.int = TRUE, 
  pval = TRUE,
  fun = "pct",
  title = "Cardiovascular conditions",
  ggtheme=custom_theme(),
  xlab = "Time (years)",
  ylab = "Survival probability",
  break.time.by = 1,
  risk.table = TRUE,
  risk.table.col = "Before_index_status_CARD",
  fontsize = 3,
  risk.table.height = 0.25,
  legend.labs = c("Only indications", "With contraindications"),
  palette = c('#BC3C29FF','#0072B5FF')
)

# Cox regression 
surv_object = Surv(time = dcm.matched$os_yrs, event = dcm.matched$dead)
fit.coxph <- coxph(surv_object ~ Before_index_status_CARD+clinspec_total+pres_total+as.factor(prac_region)+gender+age_index, data=dcm.matched)
summary(fit.coxph)
ggforest(fit.coxph, data=dcm.matched)

# testing for proportional hazards assumptions 
fit.coxph_prop = cox.zph(fit.coxph, transform = "rank")
fit.coxph_prop
ggcoxzph(fit.coxph_prop, font.main =10, font.x=8, font.y = 8)
```

