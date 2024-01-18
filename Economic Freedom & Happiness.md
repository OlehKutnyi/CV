# Economic Freedom and Happiness

## Description
Economists believe that economic freedom (EF) has a strong positive effect on subjective well-being (happiness). I decided to dig deeper and discover whether this relationship depends on a particular country. Specifically, I focus on geopolitical groups and development levels. I hoped that the the effect of increasing EF will have a stronger effect in Western Europe than in post-Soviet countries because of their history. 

## Variables
Economic freedom refers to the ability of individuals and businesses to make economic choices and decisions without government intervention or excessive restrictions. It is often used to describe the degree to which an economy allows for voluntary and market-driven economic activities to take place. Scale of measurement 1 to 100.

In turn, the happiness index is subjective, respondents are asked to rate their life satisfaction on a 1 to 10 scale.

The development level has six categories: post-Soviet, Western Europe, Sub-Saharan Africa, Latin America and Caribbean, Central and Eastern Europe, Asia and the rest of the countries in the “Other” category. Some potential categories here are omitted because of the lack of data or different factors that might have a significant effect on the dependent variable. 

Development level has three categories developed, developing, and least developed.

## Hypothesis and model
H1:The impact of economic freedom on happiness in Western Europe is stronger than in other countries.
H2: The impact of economic freedom on happiness in highly developed countries is stronger than in developing and least developed countries. 
H3: Geopolitical region has a strong effect on the relationship between economic freedom and happiness. 
H4: Development level has a strong effect on the relationship between economic freedom and happiness.

Happiness = β0 + β1(Economic Freedom) + β2(Level of Development) + β3(Geopolitical Region) + β4(Economic Freedom * Level of Development) + β5(Economic Freedom * Geopolitical Region) + ε

This model allows to assess how economic freedom impacts happiness, whether the level of development and geopolitical region have independent effects, and how the relationship between economic freedom and happiness varies across different levels of development and geopolitical culture contexts.

## Creating the data set
```R
economic_freedom_21only <- subset(economic_freeedom_2021,
                                  economic_freeedom_2021$`Index Year` == 2021)
economic_freedom_21only <- economic_freedom_21only %>%
  rename(Country_name = Name)  

world_happiness_report_2021 <- world_happiness_report_2021 %>%
  rename(Country_name = `Country name`)  

unemployment_2021<- unemployment_2021
unemployment_2021 <- unemployment_2021 %>%
  rename(unemployment21 = `2021 [YR2021]`)
unemployment_2021[162, "Country Name"] <- "Russia"

full_data <- left_join(world_happiness_report_2021, economic_freedom_21only, by = "Country_name") %>%
  left_join(unemployment_2021, by = c("Country_name" = "Country Name"))

data <- na.omit(subset(full_data, select = c("Country_name", "Regional indicator", "Ladder score", "Overall Score", "unemployment21","trade_openness_GDPshare")))
colnames(data) <- c("country_name", "regional_indicator", "happiness_index", "econ_freedom_index", "unemployment", "trade_opennes")

selected_data <- subset(data, select = c("happiness_index", "econ_freedom_index", "country_name", "unemployment", "trade_opennes", "regional_indicator"))

selected_data$regional_indicator <- factor(selected_data$regional_indicator, levels = unique(selected_data$regional_indicator))
selected_data$regional_indicator <- relevel(selected_data$regional_indicator, ref = "Sub-Saharan Africa")
levels(selected_data$regional_indicator)[levels(selected_data$regional_indicator) == "Commonwealth of Independent States"] <- "Post-Soviet"
levels(selected_data$regional_indicator)


developed_countries <- c(
  "Andorra", "Australia", "Austria", "Belgium", "Canada", "Cyprus",
  "Czech Republic", "Denmark", "Estonia", "Finland", "France", "Germany",
  "Greece", "Hong Kong SAR", "Iceland", "Ireland", "Israel", "Italy", "Japan",
  "Korea", "Latvia", "Lithuania", "Luxembourg", "Macao SAR", "Malta", "Netherlands",
  "New Zealand", "Norway", "Portugal", "Puerto Rico", "San Marino", "Singapore",
  "Slovak Republic", "Slovenia", "Spain", "Sweden", "Switzerland",
  "Taiwan Province of China", "United Kingdom", "United States")

least_developed_countries <- c(
  "Angola", "Benin", "Burkina Faso", "Burundi", "Central African Republic",
  "Chad", "Comoros", "Democratic Republic of the Congo", "Djibouti", "Eritrea",
  "Ethiopia", "Gambia", "Guinea", "Guinea-Bissau", "Lesotho", "Liberia",
  "Madagascar", "Malawi", "Mali", "Mauritania", "Mozambique", "Niger",
  "Rwanda", "Sao Tome and Principe", "Senegal", "Sierra Leone", "Somalia",
  "South Sudan", "Sudan", "Togo", "Uganda", "United Republic of Tanzania",
  "Zambia", "Afghanistan", "Bangladesh", "Bhutan", "Cambodia",
  "Lao People’s Democratic Republic", "Myanmar", "Nepal", "Timor-Leste",
  "Yemen", "Haiti", "Kiribati", "Solomon Islands", "Tuvalu")

#Creating a factor development_level
selected_data$DevelopmentLevel <- NA 

selected_data$DevelopmentLevel <- ifelse(selected_data$country_name %in% developed_countries, "Developed",
                                         ifelse(selected_data$country_name %in% least_developed_countries, "Least Developed", "Developing"))

selected_data$DevelopmentLevel <- factor(selected_data$DevelopmentLevel, levels = c("Least Developed", "Developing", "Developed"))


#now dividing by location

regional_indicators_asia <- c("East Asia", "South Asia", "Southeast Asia")
regioanl_indicators_other <- c("North America and ANZ", "Middle East and North Africa")
selected_data <- selected_data %>%
  mutate(regional_indicator = ifelse(regional_indicator %in% regional_indicators_asia, "Asia", regional_indicator))%>%
  mutate(regional_indicator = ifelse(regional_indicator %in% regioanl_indicators_other, "Other", regional_indicator))
```





