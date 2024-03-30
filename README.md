# R script used to preprocess and transform data into a longer format and then into a wider format

### Project overview
This process  involves pulling data that has been collected using an Excel template, aggregating some columns (since the data was collected using finer age disaggregated, the final export files need it to be <15yrs 15-19 yrs, 20-24 yrs and >25  yrs). After that, there is a need to unpivot it and transform it to a spread view using R libraries.

### Data sources 
Excel file: raw data collected  from the field

### Tools
- Microsoft Excel - data collection
  - RDBM merge -an add-in to merge several files
- R - Preprocess and transform [Download here](https://cran.r-project.org/bin/windows/base/R-4.3.3-win.exe)


### Data Preparation
1. Download all excel files into one folder
2. Open a new excel sheet and using the RDBM merge, merge all the files so that you are have one file containing all the dataset and save it.
3. In R load the prerequisite libraries and load the merged dataset.
4. Clean the data and transpose it

### Data preprocessing
``` R
# Clear console and define column names
remove(list = ls())
columns<-c("Site Name","Indicator", "lt1_male","lt1 _female","1_4male","1_4female","5_9male","5_9female","10_14male","10_14female","15_19male","15_19female","20_24male","20_24female",
           "25_29male","25_29female","30_34male","30_34female","35_39male","35_39female","40_44male","40_44female",
          "45_49male","45_49female","50plus_male","50plus_female")
library(readxl)
library(dplyr)
library(tidyr)
setwd("C:/Users/George.Njunge/Documents/R/Muranga/Dec 2020")

#Read Data ;single month
merged<- read_xlsx("Merged Non EMR Variance Dec20.xlsx",sheet = "Combine Sheet", skip = 2,col_names = columns)

## create DHIS age cohorts
merged<-  merged %>% replace(is.na(.), 0) %>% rowwise() %>% 
  mutate("lt15M"=sum(lt1_male,`1_4male`,`5_9male`,`10_14male`)) %>%
  mutate("lt15F"=sum(`lt1 _female`,`1_4female`,`5_9female`,`10_14female`)) %>%  mutate("15-19M"=`15_19male`) %>%
  mutate("15-19F"=`15_19female`) %>%  mutate("20-24M"=`20_24male`) %>%  mutate("20-24F"=`20_24female`) %>%
  mutate("25+M"=sum(`25_29male`,`30_34male`,`35_39male`,`40_44male`,`45_49male`,`50plus_male`)) %>%
  mutate("25+F"=sum(`25_29female`,`30_34female`,`35_39female`,`40_44female`,`45_49female`,`50plus_female`)) %>% select(-(3:26))
#unique(merged$Indicator)
View(merged)
##Transorm data to wide orientation
merged<-merged %>%
  pivot_wider(names_from = Indicator,
              values_from = c("lt15M","lt15F","15-19M","15-19F","20-24M","20-24F","25+M","25+F"),values_fill = 0)

##Reorder columns and discard unwanted
merged<- merged %>% 
  select(c(1,2,9,16,23,30,37,44,51,3,10,17,24,31,38,45,52,5,12,19,26,33,
           40,47,54,6,13,20,27,34,41,48,55,7,14,21,28,35,42,49,56,4,11,
           18,25,32,39,46,53))

#Export CSV File
write.csv(merged,"Non_EMR_Variance Dec 2020.csv")
```
### Data verification
-Inspect the new format of the data and if it ok, export the output
