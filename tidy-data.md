tidy data
================

``` r
library(tidyverse)
```

    ## -- Attaching packages ------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.0     v purrr   0.3.4
    ## v tibble  3.0.1     v dplyr   0.8.5
    ## v tidyr   1.0.3     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ---------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(dplyr)
```

## `pivot_longer`

``` r
pulse_data = haven::read_sas ("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
```

wide format to long format

``` r
pulse_data_tidy = 
  pulse_data %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  )
```

rewrite

``` r
pulse_data = 
  haven::read_sas ("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  select(id, visit,everything()) %>% 
  mutate(visit = recode(visit,"bl"="00m"))
```

## pivot wider

``` r
analysis_result = tibble(
  group = c("treatment", "treatment", "placebo", "placebo"),
  time = c("pre", "post", "pre", "post"),
  mean = c(4, 8, 3.5, 4)
)

analysis_result %>% 
  pivot_wider(
    names_from = "time",
    values_from = "mean"
  )
```

    ## # A tibble: 2 x 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

## Binding rows

Use LotR data 1. import each table

``` r
fellowship_ring = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>%
  mutate(movie = "fellowship_ring")

two_towers = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>%
  mutate(movie = "two_towers")

return_king = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>%
  mutate(movie = "return_king")
```

Bind all rows together

``` r
lotr_tidy=
  bind_rows(fellowship_ring, two_towers, return_king) %>%
  janitor::clean_names() %>% 
  select(movie,everything()) %>% 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words"
  )
```

## join dataset

``` r
pup_data = 
  read_csv("./data/FAS_pups.csv", col_types = "ciiiii") %>%
  janitor::clean_names() %>%
  mutate(sex = recode(sex, `1` = "male", `2` = "female")) 

litter_data = 
  read_csv("./data/FAS_litters.csv", col_types = "ccddiiii") %>%
  janitor::clean_names() %>%
  separate(group, into = c("dose", "day_of_tx"), sep = 3) %>%
  select(litter_number,everything()) %>%
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    dose = str_to_lower(dose))
```

join them

``` r
fas_data = 
  left_join(pup_data, litter_data, by = "litter_number") %>% 
  arrange(litter_number) %>% 
  select(litter_number,dose,day_of_tx, everything())
```
