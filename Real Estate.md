# Real Estate (R)

## Description
The project focuses on data preparation and making visualisations from the dataset about real estate in the US. It consists of:
- Status (for sale / ready to build)
- Price (in $ US dollars)
- Bed (number of bedrooms)
- Bath (number of bathrooms)
- City
- State
- House Size
- There are more variables but the project will focus on those listed above.

## Data preparation & cleaning
Downloading the data and libraries that will be used during work.
```R
realtor_data <- read.csv("realtor-data.csv")

library(ggplot)
library(tidyverse)
library(dplyr)
```
Now let's explore the data. 
```R
summary(realtor_data)
```
<img width="874" alt="Знімок екрана 2024-01-27 о 17 17 05" src="https://github.com/OlehKutnyi/CV/assets/150731232/4aae8d91-02ac-44ee-aa98-0b9f7ff2b27c">

One may already draw some conclusions:
- There are a lot of NA values.
- There are clearly some extreme values and outliers for example 99 bedrooms with an average of 3.5 or almost 200 baths with an average of 3.

Let's make a further investigation.

```R
unique(realtor_data$bed)

```
3  4  2  6  5  1  9 NA  7  8 12 13 10 11 33 24 28 14 18 20 16 15 19 17 40 21 86 31 27 42 60 22 32 99 49 29 30 23 46 36 68

```R
unique(realtor_data$bath)
```
2   1   3   5   4   7   6  NA   8   9  10  12  13  35  11  16  15  18  20  14  36  25  17  19  56  42  51  28 198  22  33  27  30  29  24  46  21

```R
unique(realtor_data$state)
"Puerto Rico"    "Virgin Islands" "Massachusetts"  "Connecticut"    "New Hampshire"  "Vermont"       
"New Jersey"     "New York"       "South Carolina" "Tennessee"      "Rhode Island"   "Virginia"      
"Wyoming"        "Maine"          "Georgia"        "Pennsylvania"   "West Virginia" 

unique(realtor_data$status)
"for_sale"       "ready_to_build"
```
```R
summary(realtor_data$price)
```
     Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0    219900    399900    709905    725000 100000000

This just proved the initial concern. Firstly, I'll delete all observations with the NA values, just as all variables that I won't use in this work. In addition, 99 bedrooms and 198 baths are considered to be an extreme event or more probably an error. Only homes with less than 10 bathrooms or bedrooms will be included in a new dataset  
I also decided to create a new variable "price_per_sq_feet".

This is pretty simple data cleaning it is better to consider techniques like IQR to remove the outliers.

```R
realtor_data_new <- realtor_data %>% 
  na.omit()%>% 
  select(status, price, bed, bath, acre_lot, city, state, house_size) %>%
  filter(bed <= 10, bath <= 10, house_size < 100000, price < 50000000) %>%
  mutate(price_per_sq_feet = price/house_size)
```

## Visualisation

**Histogram of house prices**
```R
realtor_data_new %>%
  filter(price < 2500000) %>%
  ggplot(aes(price)) +
  geom_histogram(bins = 40)
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 41 56" src="https://github.com/OlehKutnyi/CV/assets/150731232/9d122b1b-6fb0-4768-b564-3adb17393207">

**Relationship between price and size**
```R
realtor_data_new %>%
  ggplot(aes(house_size, price))+
  geom_point() +
  geom_smooth(method = "lm")
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 43 21" src="https://github.com/OlehKutnyi/CV/assets/150731232/a4699960-9ca3-42fa-a4b1-8852135a766f">
Clearly, there are still a lot of outliers and false values (very big houses with cheap prices). So I'll avoid houses larger than 20k sq feet and more expensive than $2mil.

**Average price of sq feet in each state**
```R
price_per_feet <- realtor_data_new %>%
  ggplot(aes(x = state, y = price_per_sq_feet, fill = state)) +
  geom_bar(stat = "summary", fun = "mean") +
  labs(x = "State", y = "Average Price per Sq. Feet", title = "Average Price per Sq. Feet by State") +
    theme_bw()+
    theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))+
  scale_y_continuous(labels = scales::comma)
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 45 37" src="https://github.com/OlehKutnyi/CV/assets/150731232/7e3ef898-26dc-4e9f-8fe4-6cc6318e364b">

**Average house in each state**
```R
realtor_data_new %>%
  ggplot(aes(state, house_size, fill = state)) + 
  geom_bar(stat = "summary", fun = "mean") +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1)) +
  labs(x = "State", y = "Average size of the house sold", title = "Average house size in different states")
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 47 15" src="https://github.com/OlehKutnyi/CV/assets/150731232/68ef5bbc-1558-4c03-baec-55a57dd2a5d6">

**Total values of houses in state (top 5)**
```R
realtor_data_new %>%
  group_by(state) %>%
  summarise(total_value_by_state = sum(price)) %>%
  top_n(5, total_value_by_state) %>%
  ggplot(aes(state, total_value_by_state)) +
  geom_col(fill = "blue", colour = "yellow") +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))+
  scale_y_continuous(labels = scales::comma)
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 49 51" src="https://github.com/OlehKutnyi/CV/assets/150731232/83d07626-a646-43d7-9041-d08135f6ab77">

```R
realtor_data_new %>%
  group_by(state) %>%
  summarise(total_value_by_state = sum(price)) %>%
  mutate(percentage = total_value_by_state / sum(total_value_by_state) * 100) %>%
  ggplot(aes(x = "", y = percentage, fill = state)) +
  geom_bar(width = 1, stat = "identity") +
  coord_polar("y", start = 0) +
  theme_void() +
  theme() +
  scale_fill_manual(values = rainbow(length(unique(realtor_data_new$state)))) +
  labs(fill = "State", y = "Percentage", x = NULL) +
  geom_text(aes(label = paste0(round(percentage, 1), "%")),
            position = position_stack(vjust = 0.5)) +
  guides(fill = guide_legend(title = "State")) +
  scale_y_continuous(labels = scales::percent_format())
```
<img width="1440" alt="Знімок екрана 2024-01-27 о 17 51 24" src="https://github.com/OlehKutnyi/CV/assets/150731232/09ebf7c0-8e4f-4dde-99d2-4af74bf35bce">



  




