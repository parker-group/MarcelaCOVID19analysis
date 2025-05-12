# Marcela's COVID-19 Analysis

This project explores trends in COVID-19 cases using publicly available datasets. The primary data sources include:

- [Johns Hopkins University COVID-19 Data Repository](https://github.com/CSSEGISandData/COVID-19)
- [Our World in Data COVID-19 Dataset](https://github.com/owid/covid-19-data)
- [CDC COVID Data Tracker](https://covid.cdc.gov/covid-data-tracker)
- [update and add your own]

These sources provide daily updates on cases, testing, and vaccination metrics across OC.

---

## 📊 Data Loading and Preprocessing in R

Below is an example R script used to load and clean the dataset for analysis:

```r
# Load required packages
library(tidyverse)
library(lubridate)

# Load data from Our World in Data
covid_data <- read.csv("https://covid.ourworldindata.org/data/owid-covid-data.csv")

# Filter for a specific country (e.g., Brazil)
brazil_data <- covid_data %>%
  filter(location == "Brazil") %>%
  select(date, new_cases, new_deaths, people_vaccinated) %>%
  mutate(date = as.Date(date))
```
---
## Conference Abstract 

*As of 05.03.2025* 

**Title:** Demographic Disparities and Vaccine Coverage: Predictors of SARS-CoV-2 Test Positivity in Orange County, California   

**Authors:** Marcela Lopez, Tetyana I. Vasylyeva, Daniel M. Parker  

**Background:** COVID-19, caused by the severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2), was declared a pandemic in March 2020. Vaccines against SARS-CoV-2 became available to residents in Orange County, California, in phases starting in December 2020. Previous work has identified gender, age, race, and place of residence as potential risk factors for infection.   

**Objective:** Using surveillance data, vaccination rates, and Census data, we assessed how demographic risk factors, neighborhood characteristics, and vaccination coverage influenced odds of SARS-CoV-2 infection in Orange County. We expected men, racially minoritized individuals, and individuals living in communities with greater population density to have higher odds of testing positive for COVID-19.   

**Methods:** We linked daily reported test records collated by the Orange County Health Care Agency from March 2020 to November 2021 with SARS-CoV-2 vaccination data from the California Open Data Portal and ZIP-code-level attributes from the U.S. Census Bureau (n=1,799,693). We fit generalized additive models with binomial family distributions to model test positivity as a function of individuals’ gender, age, race/ethnicity, ZIP-code population density, ZIP-code household density, the proportion of residents aged ≥65 years in each ZIP code, and age-group-specific vaccination rates – while accounting for time and location (ZIP-code geographic centroids).   

**Results:** Men had 12% increased odds of testing positive for COVID-19 compared to women (aOR: 1.12; 95% CI: 1.11, 1.13). Hispanic/Latinx individuals had over twice the odds of infection compared to Non-Hispanic White individuals (aOR: 2.19; 95% CI: 2.16, 2.22).  Those aged 80-100 years had 30% decreased odds compared to those aged 18-24 (aOR: 0.70; 95% CI: 0.68-0.72). We found an inverse relationship between test positivity and proportion of the age group vaccinated, with decreased odds of an individual testing positive after approximately 40% of their respective age group had been vaccinated. Age-stratified analyses indicated that this protective effect differed by age group, with those <65 seeing protective effects earlier than those ≥65.  

**Discussion:** Male sex and Hispanic/Latinx ethnicity were risk factors for COVID-19 infection. Even relatively moderate levels of vaccination coverage could be indirectly protective against infection. Despite earlier vaccination access, adults over 65 saw delayed protective effects, suggesting age-related differences in herd immunity.  

*350 words, excluding authors and titles*

---

## 📊 Figures

### Forest Plot for Full Model

*Version 1  - square aOR with no vertical bars for lower and upper bounds*

![Forest_plot_A](Forest_plot_3.png)  

*Version 2  - cicle aOr with with vertical bars at lower and upper bounds*

![Forest_plot_B](Forest_plot_full_model_2.png)


### Pandemic Timeline
 
The following figure is an attempt to synthesize as much information as possible. The timelines across the three levels are lined up to contextualize findings and so that comparisons can be made across time. 

![full_figure](Three_level_figure.png)


---
## Full Model

Time Period: March 2020 - November 2021  
Percent vaccinated and time are two separate variables and not included in an interaction term. 

```r
full_model =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                        s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                      + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                      + s(Percent_over_65, k = -1, bs = "ts")
                      + s(Longitude, Latitude, k = -1, bs = "sos"),
                      family = binomial, data = Final_dataset, 
                      method="fREML", select = TRUE, gamma = 1.4, nthreads=8)   
```
**Splines for Main Model**  

![full_model](Full_model_splines.png)

From this model, we can see that the percent of age group fully vaccinated is generally inversely related to the log-odds of testing positive for COVID-19. By the time a patients’ age group was over 40% vaccinated, their log odds of testing positive became less than 0. 

The spline function for week has a lot of knots and suggests a non-linear relationship between time and test results.

The downward trend observed with the spline for percent of households with ≥4 members is something that may specifically be driven by the <50 population, as the spline for the full model most closely resembles the spline for the same variable for the 18-49 age group. 

The spline for percent of ZIP code over the age of 65 has a confidence band that widens as percentage increases. Looking at the rug plot, this makese sense as there are less observations with increased percentage. The general downward trend matches the oldest age groups' decreased odds of testing positive, compared to the youngest age group. However, the confidence band includes 0 for most percentages.

ZIP code population density became positively associated with log-odds of testing positive at 9,000 people per square kilometer. Patients living in ZIP codes with densities ≥9,000 people per square kilometer had positive log-odds of testing positive.


---
## Models Stratified by Age Group

Fixed on 04.16.2025 to include entire time period (March 2020 - November 2021)  
Models contain all variables from the final model, where percent vaccinated and week are separate variables (no interaction term)

Percent vaccinated has not been lagged by any time.

**Filtering by age group:**
```r
#Filtering main dataset (named Final_dataset) into 3 groups
Group1 <- Final_dataset %>% 
  filter(Age >= 18 & Age <= 49)

Group2 <- Final_dataset %>% 
  filter(Age >= 50 & Age <= 64)

Group3 <- Final_dataset %>% 
  filter(Age >= 65)
```

**Models:**
```r
#Group 1
group1_pre_omicron =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                              s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                            + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                            + s(Percent_over_65, k = -1, bs = "ts")
                            + s(Longitude, Latitude, k = -1, bs = "sos"),
                            family = binomial, data = Group1, 
                            method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_pre_omicron))
AIC(group1_pre_omicron)

#Group 2
group2_pre_omicron =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                              s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                            + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                            + s(Percent_over_65, k = -1, bs = "ts")
                            + s(Longitude, Latitude, k = -1, bs = "sos"),
                            family = binomial, data = Group2, 
                            method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_pre_omicron))
AIC(group2_pre_omicron)

group3_pre_omicron =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                               s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                             + s(Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts") 
                             + s(Percent_over_65, k = -1, bs = "ts")
                             + s(Longitude, Latitude, k = -1, bs = "sos"),
                             family = binomial, data = Group3, 
                             method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_pre_omicron))
AIC(group3_pre_omicron)
```

AIC group 1: 589,230   
AIC group 2: 229,137.2  
AIC group 3: 153,026.1  

**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](g1_no_lag.png)


**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](g2_no_lag.png)


**Splines for Group 3: ages 65+**  

![Splines for Group 3: ages 65+](g3_no_lag.png)

After correcting the time period to include all samples from March 2020 - November 2021, the spline functions for all three age groups display similar downward trends with increasing percentage of vaccination, with coefficients ranging from 6 to -6. This downward trends is similar to the one from the full model shown above. We are no longer seeing an uptick at the end. 

For age group 1 (18-49), the spline function crosses zero at around 35%. For age group 2 (50-64%), the spline crosses zero at around 40-45%. For the eldest age group (65+), it crosses zero at around 75%. 

After discussing these findings with Tanya on Tuesday, 4.22., I'd like to explore the following question: 
*Do these percentages reflect the same period or did these groups reach these percentages at different times?* If the former, then these splines reflect a county-wide tipping point, at which all groups became protected. If the latter is true, then different age groups became protected earlier than others, reflecting potential behavioral differences or consequences of staggered vaccine administration. 

The splines for percent of households with ≥4 members differ slightly by age. While all three groups show a general upward trend, the spline for the first age group drops down again towards the end.  For two older age groups (50-64 and 65+), the splines start at around -0.5 and cross zero at around 30%. Therefore, we can conclude that OC residents living in ZIP codes where >30% of households at 4 or more members had increased odds of testing positive for COVID-19. 

The splines for proportion of ZIP code over the age of 65 and for ZIP code population density hover at around 0. Confidence bands include 0 for large sections

---
## Full Model with Vaccination Coverage Lagged by 1 month
Time period: March 2020 - November 2020  
Includes all variables from full model, except vaccination coverage has been lagged by 1 month. Model does not include the original vaccination coverage variable nor the 2-month lagged variable. 
```r
bam_1_month_lag = bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                             s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                           + s(Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts") 
                           + s(Percent_over_65, k = -1, bs = "ts")
                           + s(Longitude, Latitude, k = -1, bs = "sos"),
                           family = binomial, data = Final_dataset, 
                           method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(bam_1_month_lag))
AIC(bam_1_month_lag)

```
AIC: 976,608.1

**Splines for full model lagged by 1 month**
![Splines for 1 month lag](bam_1_month_lag.png)

---

## Models Stratified by Age Group, Vaccination Coverage Lagged by 1 month

Fixed on 04.16.2025 to include entire time period (March 2020 - November 2021)
They contain all variables from the final model, where percent vaccinated and week are separate variables (no interaction term)

Percent vaccinated has been lagged by 1 month

**Models:**
```r
#Group 1
group1_1m_lag =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group1, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_1m_lag))
AIC(group1_1m_lag)

#Group 2
group2_1m_lag =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group2, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_1m_lag))
AIC(group2_1m_lag)

#Group 3
group3_1m_lag =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                             s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                           + s(Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts") 
                           + s(Percent_over_65, k = -1, bs = "ts")
                           + s(Longitude, Latitude, k = -1, bs = "sos"),
                           family = binomial, data = Group3, 
                           method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_1m_lag))
AIC(group3_1m_lag)
```

AIC group 1: 589,013.9  
AIC group 2: 229,045.9  
AIC group 3: 152,956.1  

**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](g1_1m_lag.png)


**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](g2_1m_lag.png)


**Splines for Group 3: ages 65+**  

![Splines for Group 3: ages 65+](g3_1m_lag.png)

When delaying vaccination coverage by a month, the spline functions for vaccination coverage for all three age group shifted over to the left, with the splines crossing zero at 15%, 15%, and 65% respectively. As a check to the question posed underneath the non-lagged age-stratified spline functions, it will be good to check whether these percentages also line up with the same time period.

Of note, all three splines for vaccination coverage inflect upward at 50%, 60%, and 80%. 

---

## Full Model with Vaccination Coverage Lagged by 2 months

Time period: March 2020 - November 2020  
Includes all variables from full model, except vaccination coverage has been lagged by 2 months. Model does not include the original vaccination coverage variable nor the 1-month lagged variable. 

```r
bam_2_month_lag = bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                        s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                      + s(Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts") 
                      + s(Percent_over_65, k = -1, bs = "ts")
                      + s(Longitude, Latitude, k = -1, bs = "sos"),
                      family = binomial, data = Final_dataset, 
                      method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(bam_2_month_lag))
AIC(bam_2_month_lag)
```
AIC: 976,764.3

**Splines for full model lagged by 2 months**
![Splines for 2 month lag](bam_2_month_lag.png)


## Age Stratified Models, Lagged by 2 months

Fixed on 04.16.2025 to include entire time period (March 2020 - November 2021)
They contain all variables from the final model, where percent vaccinated and week are separate variables (no interaction term)

Percent vaccinated has been lagged by 2 months

**Models**
```r
#Group 1
group1_2m_lag =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group1, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_2m_lag))
AIC(group1_2m_lag)

#Group 2
group2_2m_lag =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                            s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                          + s(Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts") 
                          + s(Percent_over_65, k = -1, bs = "ts")
                          + s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group2, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_2m_lag))
AIC(group2_2m_lag)

#Group 3
group3_2m_lag =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity + s(Percent_hh_4_or_more, k=-1, bs = "ts") +
                             s(PopDens, k=-1, bs = "ts") + s(Week, k=-1, bs = "ts") 
                           + s(Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts") 
                           + s(Percent_over_65, k = -1, bs = "ts")
                           + s(Longitude, Latitude, k = -1, bs = "sos"),
                           family = binomial, data = Group3, 
                           method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_2m_lag))
AIC(group3_2m_lag)
```

AIC group 1: 589,185.9  
AIC group 2: 229,073.2  
AIC group 3: 152,975.1  

**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](G1_2m_lag_splines.png)


**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](G2_2m_lag_splines.png)


**Splines for Group 3: ages 65+**  

![Splines for Group 3: ages 65+](G3_2m_lag_splines.png)

---

## Full Model with Interaction Term

Time period: March 2020 - November 2020  
Models contain all variables from the final model, except that percent vaccinated and week are included as one interaction term. 

```r
interaction_only =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity  
                        + s(Week, Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts")
                        + s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                        + s(Percent_over_65, k = -1, bs = "ts")
                        + s(PopDens, k=-1, bs = "ts") 
                        + s(Longitude, Latitude, k = -1, bs = "sos"),
                        family = binomial, data = Final_dataset, 
                        method="fREML", select = TRUE, gamma = 1.4, nthreads=8)   
summary(interaction_only)
AIC(interaction_only)
```
AIC: 975,214.5

**Splines for full model with interaction term**
![Interaction_only_splines](interaction_only_splines.png)


## Models Stratified by Age Group, with Interaction Term

Time period (March 2020 - November 2021)
Models contain all variables from the final model, except that percent vaccinated and week are included as one interaction term. 
Percent vaccinated has not been lagged by any time.

**Models:**
```r
group1_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity  
                           + s(Week, Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts")                     
                            +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                            +s(Percent_over_65, k = -1, bs = "ts") 
                            +s(PopDens, k=-1, bs = "ts") 
                           +s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group1, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_int))

group2_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                           + s(Week, Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts")              
                          +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                          +s(Percent_over_65, k = -1, bs = "ts") 
                          +s(PopDens, k=-1, bs = "ts") 
                          +s(Longitude, Latitude, k = -1, bs = "sos"),
                          family = binomial, data = Group2, 
                          method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_int))

group3_int =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                           + s(Week, Percent_of_age_group_fully_vaccinated, k=-1, bs = "ts")                     
                           +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                           +s(Percent_over_65, k = -1, bs = "ts") 
                           +s(PopDens, k=-1, bs = "ts") 
                           +s(Longitude, Latitude, k = -1, bs = "sos"),
                           family = binomial, data = Group3, 
                           method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_int))

AIC(group1_int)
AIC(group2_int)
AIC(group3_int)
```

AIC group 1: 588,278.9   
AIC group 2: 228,726.1  
AIC group 3: 152,749  


**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](g1_no_lag_int.png)

**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](g2_no_lag_int.png)

**Splines for Group 3: ages 65+**   

![Splines for Group 3: ages 65+](g3_no_lag_int.png) 

---

## Models Stratified by Age Group, with Interaction Term and 1-Month Lag

Includes entire time period (March 2020 - November 2021)
Models contain all variables from the final model, except that percent vaccinated and week are included as one interaction term. 
Percent vaccinated is lagged by 1 month.

**Models:**
```r
group1_1m_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity  
                  + s(Week, Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts")            
                  +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                  +s(Percent_over_65, k = -1, bs = "ts") 
                  +s(PopDens, k=-1, bs = "ts") 
                  +s(Longitude, Latitude, k = -1, bs = "sos"),
                  family = binomial, data = Group1, 
                  method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_1m_int))

group2_1m_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                  + s(Week, Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts")                       
                  +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                  +s(Percent_over_65, k = -1, bs = "ts") 
                  +s(PopDens, k=-1, bs = "ts") 
                  +s(Longitude, Latitude, k = -1, bs = "sos"),
                  family = binomial, data = Group2, 
                  method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_1m_int))

group3_1m_int =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                   + s(Week, Percent_agegroup_fully_vaxxed_1ml, k=-1, bs = "ts")                        
                   +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                   +s(Percent_over_65, k = -1, bs = "ts") 
                   +s(PopDens, k=-1, bs = "ts") 
                   +s(Longitude, Latitude, k = -1, bs = "sos"),
                   family = binomial, data = Group3, 
                   method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_1m_int))

AIC(group1_1m_int)
AIC(group2_1m_int)
AIC(group3_1m_int)
```

AIC group 1: 588,401.4    
AIC group 2: 228,764.8  
AIC group 3: 152,743.3  


**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](g1_1m_lag_int.png)

**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](g2_1m_lag_int.png)

**Splines for Group 3: ages 65+**  

![Splines for Group 3: ages 65+](g3_1m_lag_int.png)

---

## Models Stratified by Age Group, with Interaction Term and 2-Month Lag

Includes entire time period (March 2020 - November 2021)
Models contain all variables from the final model, except that percent vaccinated and week are included as one interaction term. 
Percent vaccinated has been lagged by 2 months.

**Models:**
```r
group1_2m_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity  
                  + s(Week, Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts")                      
                  +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                  +s(Percent_over_65, k = -1, bs = "ts") 
                  +s(PopDens, k=-1, bs = "ts") 
                  +s(Longitude, Latitude, k = -1, bs = "sos"),
                  family = binomial, data = Group1, 
                  method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group1_2m_int))

group2_2m_int =  bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                  + s(Week, Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts")                           
                  +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                  +s(Percent_over_65, k = -1, bs = "ts") 
                  +s(PopDens, k=-1, bs = "ts") 
                  +s(Longitude, Latitude, k = -1, bs = "sos"),
                  family = binomial, data = Group2, 
                  method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group2_2m_int))

group3_2m_int =   bam(Positive ~ Sex + Agegroup + Race_or_ethnicity                             
                   + s(Week, Percent_agegroup_fully_vaxxed_2ml, k=-1, bs = "ts")                           
                   +s(Percent_hh_4_or_more, k=-1, bs = "ts") 
                   +s(Percent_over_65, k = -1, bs = "ts") 
                   +s(PopDens, k=-1, bs = "ts") 
                   +s(Longitude, Latitude, k = -1, bs = "sos"),
                   family = binomial, data = Group3, 
                   method="fREML", select = TRUE, gamma = 1.4, nthreads=8)  
print(summary(group3_2m_int))

AIC(group1_2m_int)
AIC(group2_2m_int)
AIC(group3_2m_int)
```

AIC group 1: 588,417.5    
AIC group 2: 228,793.5  
AIC group 3: 152,769.3  


**Splines for Group 1: ages 18 -49**  

![Splines for Group 1: ages 18 -49](g1_2m_lag_int.png)

**Splines for Group 2: ages 50-64**  

![Splines for Group 2: ages 50-64](g2_2m_lag_int.png)

**Splines for Group 3: ages 65+**  

![Splines for Group 3: ages 65+](g3_2m_lag_int.png)  


---



