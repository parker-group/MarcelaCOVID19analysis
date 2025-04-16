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

## 📈 Figures (load these as something like PNG files in the repo, then reference their location in the repo - examples below)

**Figure 1. Daily New COVID-19 Cases in Brazil**

![Daily Cases](figures/brazil_daily_cases.png)

**Figure 2. Daily New Deaths in Brazil**

![Daily Deaths](figures/brazil_daily_deaths.png)

**Figure 3. Cumulative Vaccinations in Brazil**

![Vaccination Progress](figures/brazil_vaccinations.png)

---

## 📝 Summary and Next Steps

From the figures above, we can see that Brazil experienced several waves of COVID-19 cases, with peaks aligning with increased mortality. Vaccination efforts ramped up in mid-2021, which correspond to a decline in case fatality rates.

**Next steps for Marcela:**
...

Let’s keep building out the analysis and document all updates right here in this repo!

## Age Stratified Models

Fixed on 04.16.2025 to include entire time period (March 2020 - November 2021)

```r
#Filtering main dataset into 3 groups
Group1 <- Pre_omicron_2m_lag %>% 
  filter(Age >= 18 & Age <= 49)

Group2 <- Pre_omicron_2m_lag %>% 
  filter(Age >= 50 & Age <= 64)

Group3 <- Pre_omicron_2m_lag %>% 
  filter(Age >= 65)
```

Models
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
beep(sound=2)
```

AIC group 1: 589,185.9
AIC group 2: 229,073.2
AIC group 3: 152,975.1

Code for Figures

```r
par(mfrow = c(2, 3),        # 2 rows, 3 columns
    mar   = c(4, 4, 2, 1),
    cex.main = 0.8,
    cex.lab = 0.8)

plot(group1_2m_lag, select = 4, 
     xlab = "% of Group Fully Vaccinated",
     ylab = "s(% of Group Fully Vaccinated)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "% of Group Fully Vaccinated, 2m lag",
     ylim = c(-5, 5)) 
abline(h = 0, lty = 2, col = "gray")

plot(group1_2m_lag, select = 3,
     xlab = "Week", 
     ylab = "s(Week)",
     shade = TRUE,
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Week", 
     ylim = c(-5, 5))
abline(h = 0, lty = 2, col = "gray")

plot(group1_2m_lag, select = 1, 
     xlab = "% of households with ≥4 members",
     ylab = "s(Percent of households)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "% of households in ZIP with ≥4 members", 
     ylim = c(-0.6,0.6)) 
abline(h = 0, lty = 2, col = "gray")

plot(group1_2m_lag, select = 5, 
     xlab = "Proportion of ZIP over 65",
     ylab = "s(Proportion of ZIP over 65)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Proportion of ZIP code older than 65",
     ylim = c(-0.02, 0.02)) 
abline(h = 0, lty = 2, col = "gray")

plot(group1_2m_lag, select = 2, 
     xlab = "Population Density",
     ylab = "s(Population Density)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "ZIP Code Population Density",
     ylim = c(-0.2, 0.25)) 
abline(h = 0, lty = 2, col = "gray")

plot(group1_2m_lag, select = 6, scheme =4)

#Figures for Group 2 

plot(group2_2m_lag, select = 4, 
     xlab = "% of Group Fully Vaccinated",
     ylab = "s(% of Group Fully Vaccinated)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "% of Group Fully Vaccinated, 2m lag",
     ylim = c(-5, 5)) 
abline(h = 0, lty = 2, col = "gray")

plot(group2_2m_lag, select = 3,
     xlab = "Week", 
     ylab = "s(Week)",
     shade = TRUE,
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Week", 
     ylim = c(-5, 5))
abline(h = 0, lty = 2, col = "gray")

plot(group2_2m_lag, select = 1, 
     xlab = "% of households with ≥4 members",
     ylab = "s(Percent of households)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Percent of households in ZIP with ≥4 members", 
     ylim = c(-0.75,0.6)) 
abline(h = 0, lty = 2, col = "gray")

plot(group2_2m_lag, select = 5, 
     xlab = "Proportion of ZIP over 65",
     ylab = "s(Proportion of ZIP over 65)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Proportion of ZIP code older than 65",
     ylim = c(-0.7, 0.7)) 
abline(h = 0, lty = 2, col = "gray")

plot(group2_2m_lag, select = 2, 
     xlab = "Population Density",
     ylab = "s(Population Density)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "ZIP Code Population Density",
     ylim = c(-0.005, 0.005)) 
abline(h = 0, lty = 2, col = "gray")

plot(group2_2m_lag, select = 6, scheme =4)

#Figures for Group 3 

plot(group3_2m_lag, select = 4, 
     xlab = "% of Group Fully Vaccinated",
     ylab = "s(% of Group Fully Vaccinated)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "% of Group Fully Vaccinated, 2m lag",
     ylim = c(-5, 6)) 
abline(h = 0, lty = 2, col = "gray")

plot(group3_2m_lag, select = 3,
     xlab = "Week", 
     ylab = "s(Week)",
     shade = TRUE,
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Week", 
     ylim = c(-6, 6))
abline(h = 0, lty = 2, col = "gray")

plot(group3_2m_lag, select = 1, 
     xlab = "% of households with ≥4 members",
     ylab = "s(% of households)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "% of households in ZIP with ≥4 members", 
     ylim = c(-0.75,0.75)) 
abline(h = 0, lty = 2, col = "gray")

plot(group3_2m_lag, select = 2, 
     xlab = "Population Density",
     ylab = "s(Population Density)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "ZIP Code Population Density",
     ylim = c(-0.3, 0.3)) 
abline(h = 0, lty = 2, col = "gray")

plot(group3_2m_lag, select = 5, 
     xlab = "Proportion of ZIP over 65",
     ylab = "s(Proportion of ZIP over 65)",
     shade = TRUE, 
     shade.col = "#E6F3FF", 
     rug = TRUE, 
     main = "Proportion of ZIP code older than 65",
     ylim = c(-1.2, 0.6)) 
abline(h = 0, lty = 2, col = "gray")

plot(group3_2m_lag, select = 6, scheme = 4)
```
**Splines for Group 1: ages 18 -49**  
![Splines for Group 1: ages 18 -49](G1_2m_lag_splines.png)


**Splines for Group 2: ages 50-64**  
![Splines for Group 2: ages 50-64](G2_2m_lag_splines.png)


**Splines for Group 3: ages 65+**  
![Splines for Group 3: ages 65+](G3_2m_lag_splines.png)
