# Economic Freedom and Happiness

## Description
Economists believe that economic freedom (EF) has a  positive effect on subjective well-being (happiness). I was curious whether this relationship is affected by the country's culture, which is the result of its history. I was especially interested in post-Soviet countries. The classical geopolitical groups like Western Europe, the Middle East and post-Soviet states are pretty good for this analysis. I've also added the development level to the analysis. I hoped that the effect of increasing EF would have a stronger effect in Western Europe than in post-Soviet countries because of their communist past. 

## Variables
Economic freedom refers to the ability of individuals and businesses to make economic choices and decisions without government intervention or excessive restrictions. It is often used to describe the degree to which an economy allows for voluntary and market-driven economic activities to take place. Scale of measurement 1 to 100.

In turn, the happiness index is subjective, respondents are asked to rate their life satisfaction on a 1 to 10 scale.

The geopolitical group has six categories: post-Soviet, Western Europe, Sub-Saharan Africa, Latin America and Caribbean, Central and Eastern Europe, Asia and the rest of the countries in the “Other” category. Some potential categories here are omitted because of the lack of data or different factors that might have a significant effect on the dependent variable. 

Development level has three categories developed, developing, and least developed.

|     Variable      | Values                           | Description                                                              |
|-------------------|----------------------------------|--------------------------------------------------------------------------|
| Economic freedom  | 1 - 100                          | The ability of individuals and businesses to make economic choices and   |
|                   |                                  | decisions without government intervention or excessive restrictions      |
| Happiness index   | 1 - 10                           | The happiness index is subjective, respondents are asked to rate their   |
|                   |                                  | life satisfaction on a 1 to 10 scale                                     |
| Geopolitical group| Western Europe                   |                                                                          |
|                   | post-Sovies                      |                                                                          |
|                   | Sub-Saharan Africa               |                                                                          |      
|                   | Latin America and the Caribbean  |                                                                          |
|                   | Central & Eastern Europe         |                                                                          |
|                   | Asia                             |                                                                          |
|                   | Others                           |                                                                          |
          

## Hypothesis and model
H1: The impact of economic freedom on happiness in Western Europe is stronger than in other countries.
H2: The impact of economic freedom on happiness in highly developed countries is stronger than in developing and least developed countries. 
H3: Geopolitical region has a strong effect on the relationship between economic freedom and happiness. 
H4: Development level has a strong effect on the relationship between economic freedom and happiness.

Happiness = β0 + β1(Economic Freedom) + β2(Level of Development) + β3(Geopolitical Region) + β4(Economic Freedom * Level of Development) + β5(Economic Freedom * Geopolitical Region) + ε

This model allows us to assess how economic freedom impacts happiness, whether the level of development and geopolitical region have independent effects, and how the relationship between economic freedom and happiness varies across different levels of development and geopolitical culture contexts.

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

## Some plots to feel the data

```R
plot_development <- ggplot(selected_data, aes(x = econ_freedom_index, y = happiness_index, color = DevelopmentLevel)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE, color = "blue") +
  labs(
    x = "Economic Freedom Index",
    y = "Happiness Index",
    title = "Scatter Plot with Regression Line"
  )

plot_region <- ggplot(data = selected_data,  aes(x = econ_freedom_index, y = happiness_index, color = regional_indicator)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE, color = "blue") +
  labs(
    x = "Economic Freedom Index",
    y = "Happiness Index",
    title = "Scatter Plot with Regression Line (Excluding 'other' Region)"
  )
```
<img width="1440" alt="Знімок екрана 2024-01-18 о 16 04 06" src="https://github.com/OlehKutnyi/CV/assets/150731232/92953e41-2827-40a4-93e9-9a3d87855d9b">

<img width="1440" alt="Знімок екрана 2024-01-18 о 16 04 22" src="https://github.com/OlehKutnyi/CV/assets/150731232/ef4647d6-7bb6-424b-bd58-0ea4df42bfa4">


Clearly, the developed countries are happier (which is pretty obvious), and there is also clustering based on geopolitical groups, however, it is similar to the previous graph Western Europe/Developed (basically the same countries) countries at the top and underdeveloped/Africa at the bottom. Nothing new but now I have more trust in my dataset and it once again proved the positive relationship between EF and happiness.


## Analysis

The analysis will begin with the simple model without interactions.

```R
model_no_interactions <- lm(happiness_index ~  econ_freedom_index + DevelopmentLevel+ regional_indicator, data = selected_data)
summary(model_no_interactions)
```
<img width="705" alt="Знімок екрана 2024-01-18 о 16 14 30" src="https://github.com/OlehKutnyi/CV/assets/150731232/95de123c-3432-44f7-ad47-26325b9fac35">

All variables have at least one statistically significant category (å = 0.5). Indeed both EF and development level have a positive effect on happiness. (Remember the numbers are so small because of the scale EF: 1-100; Happiness:1-10)

I will not provide a very detailed analysis here and move to the final model instead. But here is the screen shot of some other models.

<img width="470" alt="Знімок екрана 2024-01-18 о 16 24 21" src="https://github.com/OlehKutnyi/CV/assets/150731232/7f68426e-2d0b-41b9-8622-66a32f33de99">


**Final model**

```R
interact_all_model <- lm(happiness_index ~ econ_freedom_index +  DevelopmentLevel+ regional_indicator +  
                           econ_freedom_index : regional_indicator + econ_freedom_index : DevelopmentLevel, data = selected_data)
```
Unfortunately, the majority of variables are not statistically significant (model 3, screenshot above). However, it is still possible to draw some conclusions from the models above. 

When including interactions between economic freedom and regions the result suggests that economic freedom is very important for happiness in Western Europe, if compared to other regions. The interaction term is positive and statistically significant. This suggests that the positive impact of economic freedom on happiness is stronger for West European countries than for any other region of the globe.

In the second model, I did not receive any evidence suggesting that the positive effect of economic freedom depends on the level of development in my sample. This suggests that economic freedom is equally important for all countries regardless of their development level. 

When including interactions for both the geopolitical region and development level, the results do not change. The positive effect still does not depend on the level of development but preserves its variation across regional groups. Similar to the results in model 1, the impact of economic freedom on happiness is stronger for Western Europe than for other countries. 
To sum up, economic freedom is important for happiness in every country, in other words, there is a strong positive relationship between both variables. However, this effect does not depend on the country’s development level but may depend on geopolitical classification.

Despite some limitations and slightly controversial results, the analysis manages to prove two hypotheses (H1, H3) related to the regional effect. However, there is not enough evidence to accept H2 and H4, thus, one should conclude that there is no difference in the effect of economic freedom on happiness among countries with different levels of development.


### In case you want to see this a paper with all details and references use the link:
https://mpra.ub.uni-muenchen.de/119620/




