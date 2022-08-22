 # Proportion 95% confidence intervals and generation of figures

The objective of the following code is to 1) calculate 95% confidence intervals for proportions and 2) generate figures.

R version 3.6.2

## Input files
1) csv file of wrangled counts

Counts file with the following template per sheet: 

| drug cat | V1 count | V2 count | V3 count | ... | total cohort | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

2) xlsx file of data for figures 

Example template per sheet: 

| drug category | group | value | ci_lower | ci_upper | ... | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
|  |  |  |  |  |  |

3) csv files of data for heatmap figures 

Numerical heatmap proportions and prevalence tables

## Outputs

Outputs can be found in the Figures

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
# set hex values for colours
values = c('#BC3C29FF','#0072B5FF','#20854EFF','#E18727FF','#7876B1FF')


# sub-figure 1 (age, exclusion)
age_excl = read_excel("Data_figures.xlsx", sheet=1)
age_excl_1 = age_excl[which(age_excl$group == "<18 years old"), ]
age_excl_1 = age_excl_1[order(age_excl_1$value, decreasing = TRUE), ]
age_excl$facet = factor(age_excl$facet, levels=age_excl_1$facet)

plot_1 = ggplot(age_excl, aes(fill = group, y=value, x=facet)) +
  geom_bar(position="dodge", stat="identity", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=cilower, ymax=ciupper),position=position_dodge(0.601), width=0.5, colour="black",alpha=1, size=0.25) +
  ggtitle("Clinical trial exclusion rate of vulnerable age groups") +
  xlab("\nDrug category\n") +
  ylab("\nProportion (%)\n") +
  ylim(0,100) +
  labs(fill = "Legend")

plot_color_1 = plot_1 + scale_fill_manual(values) 

plot_color_1 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)

# sub-figure 2 (age, prevalence)
age_prev = read_excel("Data_figures.xlsx", sheet=2)
age_prev_1 = age_prev[which(age_prev$group == "<18 years old"), ]
age_prev_1 = age_prev_1[order(age_prev_1$value, decreasing = TRUE), ]
age_prev$facet = factor(age_prev$facet, levels=age_excl_1$facet)

plot_2 = ggplot(age_prev, aes(fill = group, y=value, x=facet)) +
  geom_bar(position="dodge", stat="identity", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=cilower, ymax=ciupper),position=position_dodge(0.601), width=0.5, colour="black",alpha=1, size=0.25) +
  ggtitle("Prevalence of vulnerable age groups") +
  xlab("\nDrug category\n") +
  ylab("\nPrevalence (%)\n") +
  ylim(0,100) +
  labs(fill = "Legend")

plot_color_2 = plot_2 + scale_fill_manual(values) 

plot_color_2 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)
)

# sub-figure 3 (clinical speciality, prevalence)
mm1 = read_excel("Data_figures.xlsx", sheet=5)
mm1_prev = mm1[which(mm1$group == "2-5 clinical specialities"), ]
mm1_prev = mm1_prev[order(mm1_prev$level, decreasing = TRUE), ]
mm1$facet = factor(mm1$facet, levels=mm2_excl$facet)

mm1$group = factor(mm1$group, levels = c("0 clinical specialities","1 clinical specialities","2-5 clinical specialities","6-10 clinical specialities","11+ clinical specialities"))

plot_5 = ggplot(mm1, aes(fill = group, y=value, x=facet)) +
  geom_bar(stat="identity", position="stack", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=ypos-sd, ymax=ypos+sd),position="identity", width=0.2, colour="black",alpha=1, size=0.25) +
  #facet_grid(rows=vars(facet)) +
  ggtitle("Prevalence of individuals by clinical speciality count") +
  xlab("\nDrug category\n") +
  ylab("\nPrevelance (%)\n") +
  ylim(0,100) +
  labs(fill = "Legend")

plot_color_5 = plot_5 + scale_fill_manual(values) 

plot_color_5 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)
)

# sub-figure 4 (clinical speciality, exclusion)
mm2 = read_excel("Data_figures.xlsx", sheet=6)
mm2_excl = mm2[which(mm2$group == "2-5 clinical specialities"), ]
mm2_excl = mm2_excl[order(mm2_excl$level, decreasing = TRUE), ]
mm2$facet = factor(mm2$facet, levels=mm2_excl$facet)

mm2$group = factor(mm2$group, levels = c("0 clinical specialities","1 clinical specialities","2-5 clinical specialities","6-10 clinical specialities","11+ clinical specialities"))

plot_6 = ggplot(mm2, aes(fill = group, y=value, x=facet)) +
  geom_bar(stat="identity", position="stack", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=ypos-sd, ymax=ypos+sd),position="identity", width=0.2, colour="black",alpha=1, size=0.25) +
  #facet_grid(rows=vars(facet)) +
  ggtitle("Clinical trial exclusion by clinical speciality count (aside from trial condition)") +
  xlab("\nDrug category\n") +
  ylab("\nProportion (%)\n") +
  ylim(0,102) +
  labs(fill = "Legend")

plot_color_6 = plot_6 + scale_fill_manual(values) 

plot_color_6 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)
)

# sub-figure 5 (medication use, exclusion)
pp1 = read_excel("Data_figures.xlsx", sheet=7)
pp1_excl = pp1[which(pp1$group == "exclusion"), ]
pp1_excl = pp1_excl[order(pp1_excl$level, decreasing = TRUE), ]
pp1$facet = factor(pp1$facet, levels=pp1_excl$facet)

plot_7 = ggplot(pp1, aes(fill = group, y=value, x=facet)) +
  geom_bar(stat="identity", position="stack", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=cilower, ymax=ciupper),position="identity", width=0.2, colour="black",alpha=1, size=0.25) +
  ggtitle("Clinical trial exclusion of medication use") +
  xlab("\nDrug category\n") +
  ylab("\nProportion (%)\n") +
  ylim(0,100) +
  labs(fill = "Legend")

plot_color_7 = plot_7 + scale_fill_manual(values) 

plot_color_7 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)
)

# sub-figure 6 (medication use, prevalence)
pp2 = read_excel("Data_figures.xlsx", sheet=8)
pp2_prev = pp2[which(pp2$group == "5-10 medications"), ]
pp2_prev = pp2_prev[order(pp2_prev$level, decreasing = TRUE), ]
pp2$facet = factor(pp2$facet, levels=pp2_prev$facet)

pp2$group = factor(pp2$group, levels = c("0 medications","1-4 medications","5-10 medications","11-20 medications","21+ medications"))

plot_8 = ggplot(pp2, aes(fill = group, y=value, x=facet)) +
  geom_bar(stat="identity", position="stack", width = 0.6) +
  geom_errorbar(aes(group=interaction(group,facet), ymin=ypos-sd, ymax=ypos+sd),position="identity", width=0.2, colour="black",alpha=1, size=0.25) +
  #facet_grid(rows=vars(facet)) +
  ggtitle("Prevalence of individuals by medication count") +
  xlab("\nDrug category\n") +
  ylab("\nPrevelance (%)\n") +
  ylim(0,102) +
  labs(fill = "Legend")

plot_color_8 = plot_8 + scale_fill_manual(values) 

plot_color_8 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
  strip.text.y = element_text(color="black", size=7, face="bold", angle = 0)
)

# sub-figure 7 (multimorbidity trend by age group)
mm3 = read_excel("Data_figures.xlsx", sheet=9)
mm3$age_group <- ordered(mm3$age_group, levels=c("<18","18-29","30-39","40-49","50-59","60-69","70-79","80+"))

plot_9 = ggplot(mm3, aes(x=factor(age_group), y=value, group=variable, color=variable, fill=variable)) +
  geom_line(size=1) +
  ggtitle("Clinical speciality multimorbidity prevalence by age group") +
  xlab("\nAge group") +
  ylab("\nPrevalence (%)") +
  ylim(0,100) +
  labs(color = "Drug category") +
  geom_point(show.legend = F) +
  guides(color=guide_legend(("Drug category"),fill=FALSE))

plot_color_9 = plot_9 + scale_color_manual(values = c('#BC3C29FF','#0072B5FF','#20854EFF','#E18727FF','#7876B1FF','#6F99ADFF','#FFDC91FF','#EE4C97FF','#0072B599','#BC3C2999','#20854E99','#E1872799','#7876B199')) 

plot_color_9 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
)

# sub-figure 8 (polypharmacy trend by age group)
pp3 = read_excel("Data_figures.xlsx", sheet=10)
pp3$age_group <- ordered(pp3$age_group, levels=c("<18","18-29","30-39","40-49","50-59","60-69","70-79","80+"))

plot_10 = ggplot(pp3, aes(x=factor(age_group), y=value, group=variable, color=variable, fill=variable)) +
  geom_line(size=1) +
  ggtitle("Polypharmacy prevalence by age group") +
  xlab("\nAge group") +
  ylab("\nPrevalence (%)") +
  ylim(0,100) +
  labs(color = "Drug category") +
  geom_point(show.legend = F) +
  guides(color=guide_legend(("Drug category"),fill=FALSE))

plot_color_10 = plot_10 + scale_color_manual(values = c('#BC3C29FF','#0072B5FF','#20854EFF','#E18727FF','#7876B1FF','#6F99ADFF','#FFDC91FF','#EE4C97FF','#0072B599','#BC3C2999','#20854E99','#E1872799','#7876B199')) 

plot_color_10 + theme(
  plot.title = element_text(color="black",size=14, face="bold", hjust=0.5),
  axis.title.x = element_text(color="black", size=14, face="bold"),
  axis.title.y = element_text(color="black", size=14, face="bold"),
  axis.text.x = element_text(color="black", size=8, face="bold", hjust=0.5, vjust = 0.5),
  axis.text.y = element_text(color="black", size=7, face="bold", hjust=0.5, vjust = 0.5),
  legend.title = element_text(color="black", size=10, face="bold", hjust = 0.5),
  panel.grid.major.x = element_blank(),
  axis.line = element_line(color = "black"),
)

# heatmap example (exclusion)
clin_trials_main = read.csv('HM_clin_trials_clust_main.csv', row.names = 1)
clin_trials_main_nums = clin_trials_main

clin_trials_main[] = lapply(clin_trials_main, function(x) ifelse(x>=50, "50-100", 
                                                                 ifelse(x >= 30 & x<50, "30-49.9",
                                                                        ifelse(x >= 20 & x <30, "20-29.9",
                                                                               ifelse(x <20,"<20",x)))))

colour_names_2 = setNames(c('#BC3C29FF','#E18727FF','#FFDC91FF','#20854E99'), c("50-100","30-49.9","20-29.9","<20"))

pdf("hm_clin_all_main.pdf",width=11.7,height=8.3,paper="a4r")
ht_2 = Heatmap(clin_trials_main, name = "Exclusion proportion (%)", col = colour_names_2,
               column_title = "Drug category",
               column_title_side = "bottom",
               column_title_gp = gpar(fontsize = 10, fontface = "bold"),
               row_title = "Clinical speciality",
               row_title_side = "left",
               row_title_rot = 0,
               row_title_gp = gpar(fontsize = 10, fontface = "bold"),
               cell_fun = function(j, i, x, y, width, height, fill) {
                 grid.text(sprintf("%.1f", clin_trials_main_nums[i, j]), x, y, gp = gpar(fontsize = 8))
               },
               rect_gp = gpar(col = "white", lwd = 2),
               width = unit(12, "cm"), height = unit(15, "cm"),
               column_names_gp = gpar(fontsize=7,  fontface = "bold"),
               row_names_side = "left",
               row_names_gp = gpar(fontsize=7,  fontface = "bold"))
draw(ht_2)
dev.off()

# heatmap example (prevalence)
rwe_all_mirror = read.csv('HM_RWE_all_mirror.csv', row.names = 1)
rwe_all_mirror_nums = rwe_all_mirror
rwe_all_mirror[] = lapply(rwe_all_mirror, function(x) ifelse(x>=10, "10+", 
                                                             ifelse(x >= 5 & x<10, "5-9.9",
                                                                    ifelse(x >= 1 & x <5, "1-4.9",
                                                                           ifelse(x <1,"<1",x)))))

colour_names_3 = setNames(c('#BC3C29FF','#E18727FF','#FFDC91FF','#20854E99'), c("10+","5-9.9","1-4.9","<1"))

pdf("hm_rwe_all_mirror.pdf",width=11.7,height=8.3,paper="a4r")
ht_3 = Heatmap(rwe_all_mirror, name = "Prevalence (%)", col = colour_names_3,
               column_title = "Drug category",
               column_title_side = "bottom",
               column_title_gp = gpar(fontsize = 10, fontface = "bold"),
               row_title = "Clinical speciality",
               row_title_side = "left",
               row_title_rot = 0,
               row_title_gp = gpar(fontsize = 10, fontface = "bold"),
               cell_fun = function(j, i, x, y, width, height, fill) {
                 grid.text(sprintf("%.1f", rwe_all_mirror_nums[i, j]), x, y, gp = gpar(fontsize = 8))
               },
               rect_gp = gpar(col = "white", lwd = 2),
               width = unit(12, "cm"), height = unit(15, "cm"),
               column_names_gp = gpar(fontsize=7,  fontface = "bold"),
               row_names_side = "left",
               row_names_gp = gpar(fontsize=7,  fontface = "bold"))
draw(ht_3)
dev.off()
```