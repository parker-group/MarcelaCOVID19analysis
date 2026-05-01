# Demographic Disparities and Vaccine Coverage: Predictors of SARS-CoV-2 Test Positivity in Orange County, California, USA
Marcela Lopez<sup>1</sup>, Tetyana I. Vasylyeva<sup>2</sup>, Matthew Zahn<sup>3</sup>, Daniel M. Parker<sup>1,2</sup>

###AUTHOR AFFILIATIONS
1. Department of Epidemiology and Biostatistics, University of California, Irvine, California, USA 
2. Department of Population Health and Disease Prevention, University of California, Irvine, California, USA
3. Orange County Health Care Agency

###ABSTRACT 
Background: While previous studies have documented demographic disparities in COVID-19 incidence in the United States, it remains unclear how these disparities evolved with the emergence of new variants, the introduction of vaccines, and changing levels of population immunity. Methods: We analyzed 1.8 million SARS-CoV-2 RT-PCR tests reported in Orange County, California, USA, between March 2020 and November 2021 to evaluate how vaccination coverage and demographic factors influenced infection risk. We fit a generalized additive model (GAM) with a binomial family distribution to model test positivity as a function of individuals’ sex, age, race/ethnicity, ZIP code-level attributes, and age-group-specific vaccination coverage, while accounting for time (epidemiological weeks) and location. Given the staggered vaccine roll-out by age group, we explored whether vaccination coverage impacted each age group differently. Results: Hispanic/Latinx individuals had greater than twice the odds of diagnosed infection (aOR: 2.19; 95% CI: 2.16-2.22) compared to non-Hispanic white individuals. Increasing age-specific vaccination coverage was strongly associated with reduced test positivity. However, despite being prioritized for early vaccination, reductions in test positivity among older adults were not apparent until vaccination coverage increased across younger age groups. Discussion: Persistent spatial disparities in test positivity, even after adjustment for demographic and ZIP code-level factors, indicate the continued importance of local structural and contextual drivers of infection risk. Population-level benefits of vaccination may depend not only on coverage within high-risk groups, but also on vaccination uptake among more socially active age groups.  

---
## Table of Contents 
1. Data Sources [here](#data-sources)
2. Data Processing in R [here](#data-processing-in-r)

---

## Data Sources
**Orange County Health Care Agency:**  Test Result Data from RT-PCR tests conducted in Orange County from 2020-2021


**COVID-19 Vaccine Progress Dashboard Data:** Vaccination Coverage by ZIP Code and Age Group (downloaded from the California Open Data Portal in June 2023 (available at: https://data.ca.gov/dataset/covid-19-vaccineprogress-dashboard-data))

**U.S. Census Bureau’s 2021 American Community Survey:** ZIP-code-level characteristics
---

** 
## 📊 Data Loading and Preprocessing in R

Below is R script used to load and clean the dataset for analysis:

```r
# Load required packages
library(lubridate)
library(dplyr)
library(fastmap)
library(zipcodeR)
library(tidyr)
library(zoo)
library(mgcv)

#Filtering data ------

#Filtering ZIP, Start Date, Sex, and Test Results
Raw_data_2 <- Raw_data %>%
  filter(Zip %in% OCzipcodes$Zip) 

Raw_data_3 <- Raw_data_2 %>% 
  filter(Specimen.Collected.Date >= "2020-03-03") %>%
  filter(Sex %in% c('M', 'F')) 

All_negatives <- Raw_data_3 %>%
  filter(raw_data_3$TestResult %in% c('Negative'))

All_positives <- Raw_data_3 %>%
  filter(TestResult == 'Positive') 

First_positives <- All_positives %>% 
  arrange(IncidentID, Specimen.Collected.Date) %>%
  group_by(IncidentID) %>%
  slice(1) %>% 
  ungroup 

Negatives_and_first_positives <- bind_rows(All_negatives, First_positives)
Negatives_and_first_positives$Positive <- ifelse(Negatives_and_first_positives$TestResult == 'Positive', 1, 0)  

#Combining Race and Ethnicity into One Variable 
Negatives_and_first_positives$Ethnicity <- ifelse(is.na(Negatives_and_first_positives$Ethnicity), "Unknown", Negatives_and_first_positives$Ethnicity)

Negatives_and_first_positives$Race_or_ethnicity <- ifelse(is.na(Negatives_and_first_positives$Race), 10,
                                                          ifelse(Negatives_and_first_positives$Race == 'White', 1,
                                                                 ifelse(Negatives_and_first_positives$Race == 'Asian', 2,
                                                                        ifelse(Negatives_and_first_positives$Race == 'Black or African American', 3,
                                                                               ifelse(Negatives_and_first_positives$Race == 'American Indian or Alaska Native', 4,
                                                                                      ifelse(Negatives_and_first_positives$Race == 'Native Hawaiian or Other Pacific Islander', 5, 10))))))
Negatives_and_first_positives$Race_or_ethnicity <- 
  ifelse(Negatives_and_first_positives$Ethnicity == 'Hispanic or Latino', 6, Negatives_and_first_positives$Race_or_ethnicity) 

#Creating Age Groups
Negatives_and_first_positives <- negatives_and_first_positives %>%
  mutate(Agegroup = case_when(
    Age >= 0 & Age < 18 ~ 1,
    Age >= 18 & Age < 25 ~ 2,
    Age >= 25 & Age < 30 ~ 3,
    Age >= 30 & Age < 35 ~ 4,
    Age >= 35 & Age < 40 ~ 5,
    Age >= 40 & Age < 50 ~ 6,
    Age >= 50 & Age < 60 ~ 7,
    Age >= 60 & Age < 70 ~ 8,
    Age >= 70 & Age < 80 ~ 9,
    Age >= 80 & Age <= 100 ~ 10,
    is.na(Age) | Age > 100 ~ 11,
    TRUE ~ NA_real_)) 

#Obtaining ZIP Code Centroid Latitude and Longitude for our 85 ZIP Codes of Interest
OCzipcodes$Latitude = geocode_zip(OCzipcodes$Zip)$lat
OCzipcodes$Longitude = geocode_zip(OCzipcodes$Zip)$lng

Negatives_and_first_positives_2 = merge(Negatives_and_first_positives, OCzipcodes, by = "Zip", all.x = TRUE, default=0)


Negatives_and_first_positives_2$Day_of_week <- as.integer(format(negatives_and_first_positives_2$Specimen.Collected.Date, "%u"))
Negatives_and_first_positives_2$Days_to_most_recent_tuesday <- (negatives_and_first_positives_2$Day_of_week - 2) %% 7
Negatives_and_first_positives_2$Tuesday <- negatives_and_first_positives_2$Specimen.Collected.Date - negatives_and_first_positives_2$Days_to_most_recent_tuesday
Negatives_and_first_positives_2$Week = as.integer(negatives_and_first_positives_2$Tuesday-Start_date)/7

#Merging ZIP Code Vaccine Data 
Vaccine_data_by_ZIP <- Vaccine_data_by_ZIP %>%
  filter(county == 'Orange' & as_of_date <= "2022-06-07") %>% 
  rename(Tuesday = as_of_date, Zip = zip_code_tabulation_area) 
Vaccine_data_by_ZIP$Mergezip = paste(Vaccine_data_by_ZIP$Tuesday, Vaccine_data_by_ZIP$Zip, sep = "-")
Vaccine_data_by_ZIP_2 <- Vaccine_data_by_ZIP %>% 
  select(Mergezip, Zip, Tuesday, vaccine_equity_metric_quartile, Tuesday, percent_of_population_fully_vaccinated, percent_of_population_with_1_plus_dose)

Negatives_and_first_positives_2$Mergezip <- paste(Negatives_and_first_positives_2$Tuesday, Negatives_and_first_positives_2$Zip, sep = "-")
Negatives_and_first_positives_3 <- merge(Negatives_and_first_positives_2, Vaccine_data_by_ZIP_2, by = "Mergezip", all.x = TRUE, default=0)

#Merging Age Group Vaccine Data
Vaccine_data_by_age %>%
  filter(administered_date <= "2022-06-07") %>% 
  rename(Specimen.Collected.Date = administered_date) 

Vaccine_data_by_age$Percent_of_age_group_fully_vaccinated <- 
  (Vaccine_data_by_age$cumulative_fully_vaccinated / Vaccine_data_by_age$est_population)

Vaccine_data_by_age$Percent_of_age_group_partially_vaccinated <- 
  (Vaccine_data_by_age$cumulative_at_least_one_dose/ Vaccine_data_by_age$est_population)

Vaccine_data_by_age$Mergeage <- 
  paste(Vaccine_data_by_age$Specimen.Collected.Date, Vaccine_data_by_age$demographic_value, sep = "-")

Vaccine_data_by_age_2 <- Vaccine_data_by_age %>%
  select(Mergeage, Specimen.Collected.Date,demographic_value, 
         Percent_of_age_group_fully_vaccinated, Percent_of_age_group_partially_vaccinated)


Negatives_and_first_positives_3 <- Negatives_and_first_positives_3 %>%
  mutate(Demographic_value = case_when(
    Age >= 18 & Age <= 49 ~ '18-49',
    Age >= 50 & Age <= 64 ~ '50-64',
    Age >= 65 ~ '65+',
    TRUE ~ NA_character_
  ),
  Age_vacc_binary = case_when(
    Specimen.Collected.Date >= "2021-01-13" & Age >= 65 ~ 1,  
    Specimen.Collected.Date >= "2021-04-01" & Age >= 50 ~ 1,
    Specimen.Collected.Date >= "2021-04-15" & Age >= 16 ~ 1,
    TRUE ~ 0
  )) 

Negatives_and_first_positives_3$Mergeage <- paste(Negatives_and_first_positives_3$Specimen.Collected.Date, Negatives_and_first_positives_3$Demographic_value, sep = "-")

Negatives_and_first_positives_4 = merge(Negatives_and_first_positives_3, Vaccine_data_by_age_2, by = "Mergeage", all.x = TRUE, default=0)


Negatives_and_first_positives_5 <- Negatives_and_first_positives_4 %>% 
  filter(Agegroup > 1 & Agegroup < 11 & Race_or_ethnicity < 7) 


#Adding Census Data
Negatives_and_first_positives_6 <- merge(Negatives_and_first_positives_5, OCzipcodes_and_Census_Data, by = "Zip", all.x = TRUE)

Pre_omicron <- Negatives_and_first_positives_6 %>% 
  filter(Specimen.Collected.Date <= as.Date("2021-11-30"))

#Factorizing variables
Pre_omicron$Sex <- as.factor(Pre_omicron$Sex)
Pre_omicron$Agegroup <- as.factor(Pre_omicron$Agegroup)
Pre_omicron$Race_or_ethnicity <- as.factor(Pre_omicron$Race_or_ethnicity)

#Groups 
Group1 <- Pre_omicron %>% 
  filter(Age >= 18 & Age <= 49) 
Group2 <- Pre_omicron %>% 
  filter(Age >= 50 & Age <= 64) 
Group3 <- Pre_omicron %>% 
  filter(Age >= 65)

```
## MODELS

###MAIN MODEL 
```r
main_model =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
              + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
              + s(Percent_over_65, k = -1, bs = "ts")
              + s(Longitude, Latitude, k = -1, bs = "sos"), random = list(IncidentID = ~1),
              family = binomial, data = Pre_omicron, 
              method="fREML", select = TRUE, gamma = 1.4, nthreads=8) 

###AGE STRATIFIED MODELS

Group1_model =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"), random = list(IncidentID = ~1),
                          family = binomial, data = Group1, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  

Group2_model =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"), random = list(IncidentID = ~1),
                          family = binomial, data = Group2, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  

Group3_pre_omicron =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                             s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                           + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                           + s(Percent_over_65, k = -1, bs = "ts")
                           + s(Longitude, Latitude, k = -1, bs = "sos"), random = list(IncidentID = ~1),
                           family = binomial, data = Group3, 
                           method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  

###BINNED MODELS 

#Group1

Group1$Vaccination_bins <- relevel(as.factor(Group1$Vaccination_bins), ref = "30-35%")

group1_binned =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                       s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                     + s(Percent_over_65, k = -1, bs = "ts")
                     + s(Longitude, Latitude, k = -1, bs = "sos")
                     + Vaccination_bins,
                     family = binomial, data = Group1, 
                     method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  

#Group 2 
Group2$Vaccination_bins <- relevel(as.factor(Group2$Vaccination_bins), ref = "45-50%")
group2_binned =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                       s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                     + s(Percent_over_65, k = -1, bs = "ts")
                     + s(Longitude, Latitude, k = -1, bs = "sos")
                     + Vaccination_bins,
                     family = binomial, data = Group2, 
                     method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
#Group 3
Group3$Vaccination_bins <- relevel(as.factor(Group3$Vaccination_bins), ref = "75-80%")
group3_binned =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                        s(Population_density, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                      + Percent_of_age_group_fully_vaccinated
                      #   + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                      + s(Percent_over_65, k = -1, bs = "ts")
                      + s(Longitude, Latitude, k = -1, bs = "sos")
                      + Vaccination_bins,
                      family = binomial, data = Group3, 
                      method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  

```

