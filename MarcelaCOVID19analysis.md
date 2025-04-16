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
