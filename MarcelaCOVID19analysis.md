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

## 📝 Summary and Next Steps

From the figures above, we can see that Brazil experienced several waves of COVID-19 cases, with peaks aligning with increased mortality. Vaccination efforts ramped up in mid-2021, which correspond to a decline in case fatality rates.

**Next steps for Marcela:**
...

Let’s keep building out the analysis and document all updates right here in this repo!

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
                      family = binomial, data = Pre_omicron_2m_lag, 
                      method="fREML", select = TRUE, gamma = 1.4, nthreads=8)   
```
**Splines for Main Model**  

![full_model](Full_model_splines.png)

From this model, we can see that the percent of age group fully vaccinated is generally inversely related to the log-odds of testing positive for COVID-19. By the time a patients’ age group was over 40% vaccinated, their log odds of testing positive became less than 0. 

The spline function for week has a lot of knots and suggests a non-linear relationship between time and test results.

ZIP code population density became positively associated with log-odds of testing positive at 9,000 people per square kilometer. Patients living in ZIP codes with densities ≥9,000 people per square kilometer had positive log-odds of testing positive.

---
## Models Stratified by Age Group

Fixed on 04.16.2025 to include entire time period (March 2020 - November 2021)  
Models contain all variables from the final model, where percent vaccinated and week are separate variables (no interaction term)

Percent vaccinated has not been lagged by any time.

**Filtering by age group:**
```r
#Filtering main dataset (named Pre_omicron_2m_lag) into 3 groups
Group1 <- Pre_omicron_2m_lag %>% 
  filter(Age >= 18 & Age <= 49)

Group2 <- Pre_omicron_2m_lag %>% 
  filter(Age >= 50 & Age <= 64)

Group3 <- Pre_omicron_2m_lag %>% 
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

---

## Models Stratified by Age Group, Lagged by 1 month

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

---

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

## Models Stratified by Age Group, with Interaction Term

Includes entire time period (March 2020 - November 2021)
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

