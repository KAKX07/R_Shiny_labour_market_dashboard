# ==============================================================================
# data_prep.R
# European Labour Market Dashboard — Data Preparation
# Employment unit is per 1000
# ==============================================================================

#Library

library(tidyverse)
library(dplyr)
library(ggplot2)
library(knitr)

# ==============================================================================
# 0. Constants
# ==============================================================================

#European OECD countries — used to filter all datasets
europe_iso <- c("AUT","BEL","CZE","DNK","EST","FIN","FRA","DEU",
                "GRC","HUN","ISL","IRL","ITA","LVA","LTU","LUX",
                "NLD","NOR","POL","PRT","SVK","SVN","ESP","SWE",
                "CHE","GBR","BGR","HRV","ROU")

#Mutually exclusive macro-economic sectors (NACE Rev.2)
#Prevents double counting — sector C is already included in BTE
secteurs_exclusifs <- c("A", "BTE", "F", "GTI", "J", "K", "L", "M_N", "OTQ", "RTU")

#Short English labels for each sector code
sector_translation <- c(
  "A"   = "Agriculture",
  "BTE" = "Industry",
  "F"   = "Construction",
  "GTI" = "Trade & Transport",
  "J"   = "ICT",
  "K"   = "Finance",
  "L"   = "Real Estate",
  "M_N" = "Professional Services",
  "OTQ" = "Public Administration",
  "RTU" = "Arts & Recreation"
)

#Short English labels for each skill category code
skill_translation <- c(
  "01" = "Arts & Humanities",
  "02" = "Attitudes",
  "03" = "Business Processes",
  "04" = "Cognitive Skills",
  "05" = "Communication",
  "06" = "Digital Skills",
  "07" = "Law & Security",
  "08" = "Medical Knowledge",
  "09" = "Physical Abilities",
  "10" = "Production & Technology",
  "11" = "Resource Management",
  "12" = "Scientific Knowledge",
  "13" = "Social Skills",
  "14" = "Teaching & Training"
)

# ==============================================================================
# 1. Load and transform source data
# ==============================================================================

#Load employment data — sum M + F to get total per country / age group / year
data_emp <- read_csv("OECD_EMP.csv") %>%
  select(
    ISO         = REF_AREA,
    Country     = `Zone de référence`,
    Gender      = SEX,
    Age         = AGE,
    Period      = TIME_PERIOD,
    Employement = OBS_VALUE
  )

total_data_emp <- data_emp %>%
  group_by(ISO, Country, Age, Period) %>%
  summarise(Employement = sum(Employement, na.rm = TRUE), .groups = "drop") %>%
  filter(ISO %in% europe_iso, Period != 2025)

total_data_emp

#Load unemployment data
data_unemp <- read_csv("OECD_UNEMP.csv") %>%
  select(
    ISO           = REF_AREA,
    Country       = `Zone de référence`,
    Age           = AGE,
    Period        = TIME_PERIOD,
    Unemployement = OBS_VALUE
  ) %>%
  filter(ISO %in% europe_iso, Period != 2025)

data_unemp

#Load average wage data
data_avg_wage <- read_csv("OECD_avg_wage.csv") %>%
  select(
    ISO      = REF_AREA,
    Country  = `Zone de référence`,
    Period   = TIME_PERIOD,
    AVG_WAGE = OBS_VALUE
  ) %>%
  filter(ISO %in% europe_iso, Period != 2025)

data_avg_wage

#Load AI adoption data
#We filter with BUS_TOTAL because it represents the total value across all firm sizes
data_ai <- read_csv("OECD_AI_adoption.csv") %>%
  filter(`V7 Breakdowns` == "BUS_TOTAL") %>%
  select(
    ISO     = `Country ISO3`,
    Country = `Zone de référence`,
    Period  = `Période temporelle`,
    pct_AI  = Value
  ) %>%
  arrange(Country, Period) %>%
  filter(ISO %in% europe_iso)

data_ai

#Load employment per sector data
#We filter on mutually exclusive NACE sectors to avoid double counting
#Sector labels are translated to short English names for readability
data_emp_sector <- read_csv("OECD_EMP_PER_SECTOR.csv") %>%
  filter(TRANSACTION == "EMP") %>%
  filter(ACTIVITY %in% secteurs_exclusifs) %>%
  select(
    ISO          = REF_AREA,
    Country      = `Zone de référence`,
    Sector       = ACTIVITY,
    Sector_label = `Activité économique`,
    Period       = TIME_PERIOD,
    Employment   = OBS_VALUE
  ) %>%
  filter(ISO %in% europe_iso, Period != 2025) %>%
  filter(!is.na(Employment)) %>%
  mutate(Sector_label = sector_translation[Sector])  #replace French long labels

data_emp_sector

#Load OECD Skills for Jobs data
#Shortage indicator : positive = skill shortage / negative = skill surplus
#Skill labels translated to short English names to fix UTF-8 encoding issues

data_skills <- read_csv("OECD_JOBS_NEED.csv", locale = locale(encoding = "UTF-8")) %>%
  select(
    ISO        = LOCATION,
    Country    = Pays,
    Skill_Code = SKILL,
    Skill      = `Compétences`,
    Shortage   = OBS_VALUE
  ) %>%
  filter(ISO %in% europe_iso) %>%
  filter(!grepl("-", Skill_Code)) %>%  #keep only 14 main categories, not subcategories
  mutate(Skill = skill_translation[Skill_Code]) %>%  #replace French encoded labels
  arrange(Country, Shortage)

data_skills

# ==============================================================================
# 2. Derived indicators
# ==============================================================================

#Compute unemployment rate per country / age group / year

data_labour <- total_data_emp %>%
  left_join(data_unemp, by = c("ISO", "Country", "Age", "Period")) %>%
  mutate(
    Unemployment_Rate = (Unemployement / (Employement + Unemployement)) * 100
  )

data_labour

#Compute year-over-year employment growth rate per country / age group
data_emp_growth <- total_data_emp %>%
  group_by(ISO, Country, Age) %>%
  arrange(Period) %>%
  mutate(
    Emp_Growth = (Employement - lag(Employement)) / lag(Employement) * 100
  ) %>%
  ungroup()

data_emp_growth

#Compute 15-24 and 55-64 employment share of total employment
#These are indicators of youth precarity and active ageing
data_emp_age_share <- total_data_emp %>%
  group_by(ISO, Country, Period) %>%
  mutate(Total_Emp = sum(Employement, na.rm = TRUE)) %>%
  ungroup() %>%
  mutate(
    Age_Share = Employement / Total_Emp * 100,
    Age = recode(Age,                           #rename age codes to readable labels
                 "Y15T24" = "15-24 years",
                 "Y25T54" = "25-54 years",
                 "Y55T64" = "55-64 years"
    )
  )

data_emp_age_share

#Compute wage growth rate (YoY) and wage index base 100 in 2000
#Wage index allows cross-country comparison by neutralizing absolute wage differences
data_avg_wage <- data_avg_wage %>%
  group_by(ISO, Country) %>%
  arrange(Period) %>%
  mutate(
    Wage_Growth = (AVG_WAGE - lag(AVG_WAGE)) / lag(AVG_WAGE) * 100,
    Wage_Index  = AVG_WAGE / AVG_WAGE[Period == 2000][1] * 100  #base 100 in 2000, [1] for safety
  ) %>%
  ungroup()

data_avg_wage

#Compute self employment share per country / year
#We reconstruct the total by summing mutually exclusive sectors
data_emp_self <- read_csv("OECD_EMP_PER_SECTOR.csv") %>%
  filter(TRANSACTION %in% c("EMP", "SELF")) %>%
  filter(ACTIVITY %in% secteurs_exclusifs) %>%  #same sectors as data_emp_sector
  select(
    ISO        = REF_AREA,
    Country    = `Zone de référence`,
    Type       = TRANSACTION,
    Period     = TIME_PERIOD,
    Employment = OBS_VALUE
  ) %>%
  filter(ISO %in% europe_iso) %>%
  filter(!is.na(Employment)) %>%
  group_by(ISO, Country, Type, Period) %>%
  summarise(Employment = sum(Employment, na.rm = TRUE), .groups = "drop") %>%
  pivot_wider(names_from = Type, values_from = Employment) %>%
  mutate(Self_Employment_Share = SELF / EMP * 100)

data_emp_self

# ==============================================================================
# 3. Final join : AI impact on labour market (core of page 2)
# ==============================================================================

#Aggregate employment growth across all age groups before joining
#This avoids the approximation of filtering on a single age group
data_emp_growth_15_64 <- total_data_emp %>%
  group_by(ISO, Country, Period) %>%
  summarise(Employement_Total = sum(Employement, na.rm = TRUE), .groups = "drop") %>%
  group_by(ISO, Country) %>%
  arrange(Period) %>%
  mutate(
    Emp_Growth = (Employement_Total - lag(Employement_Total)) / lag(Employement_Total) * 100
  ) %>%
  ungroup()

data_emp_growth_15_64

#Join AI adoption with wage growth and employment growth
#Common period between AI and labour datasets : 2017-2024
data_ai_labour <- data_ai %>%
  left_join(data_avg_wage %>% select(ISO, Country, Period, AVG_WAGE, Wage_Growth),
            by = c("ISO", "Country", "Period")) %>%
  left_join(data_emp_growth_15_64 %>% select(ISO, Country, Period, Emp_Growth),
            by = c("ISO", "Country", "Period")) %>%
  drop_na(pct_AI, AVG_WAGE)

data_ai_labour

# ==============================================================================
# 4. Save all prepared data for the Shiny app
# ==============================================================================

save(
  total_data_emp,
  data_unemp,
  data_avg_wage,
  data_ai,
  data_emp_sector,
  data_skills,
  data_labour,
  data_emp_growth,
  data_emp_age_share,
  data_emp_self,
  data_emp_growth_15_64,
  data_ai_labour,
  file = "data_prepared.RData"
)

message("data_prep.R done — data_prepared.RData saved")