 # Proportion 95% confidence intervals and generation of figures

The objective of the following code is to 1) calculate 95% confidence intervals for proportions and 2) generate figures.

R version 3.6.2

## Input files
1) csv file of wrangled counts

Counts file with the following template per sheet: 

| drug cat | V1 count | V2 count | V3 count | ... | total cohort | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

## Importing packages 
```R
library(purrr)
library(readxl)
library(ggplot2)
library(fmsb)
library(ggsci)
library(ComplexHeatmap)
library(grid)
```
 
## Calculating confidence intervals for proportions 
```R
# example: to calculate 95% CIs for proportion using bootstrap percentile method with 5 variables for 12 drug categories 
output = Map(prop.test, x = CI_age_excl$v1, n = CI_age_excl$total)
output2 = Map(prop.test,x=CI_age_excl$clin1, n=CI_age_excl$Total)
output3 = Map(prop.test,x=CI_age_excl$clin2, n=CI_age_excl$Total)
output4 = Map(prop.test,x=CI_age_excl$clin3, n=CI_age_excl$Total)
output5 = Map(prop.test,x=CI_age_excl$clin4, n=CI_age_excl$Total)

output_list = list(output1, output2, output3, output4, output5)

output_outcome = lapply(output_list, function(df){
  df_list = list(df[[1]][[6]],df[[2]][[6]],df[[3]][[6]],df[[4]][[6]],df[[5]][[6]],df[[6]][[6]],df[[7]][[6]],df[[8]][[6]],df[[9]][[6]],df[[10]][[6]],df[[11]][[6]],df[[12]][[6]],df[[13]][[6]])
})

lapply(output_outcome, function(x) write.table(data.frame(x), '95_CI_age_excl.csv'  , append= T, sep=','))

# to verify for qc, following function can be used for individual proportion testing 
prop.test("proportion x", "total")

# repeat and format as necessary for other sheets 
```
## Plotting figures 
```R



```