Scrape MSU Counts
================

This document contains R code to download daily case totals reported
from the COVID-19 Dashboard by the Center for Systems Science and
Engineering (CSEE) at Johns Hopkins University and combines that with
information obtained on weekly totals of cases at Montana State
University

### Download Daily County Data

``` r
data_in <- read_csv('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_US.csv') %>% 
  filter(FIPS == 30031) %>% 
  select(-UID, -iso2, -iso3, -FIPS, -Admin2, 
                -code3, -Country_Region, -Lat, -Long_, -Province_State) %>% 
  pivot_longer(cols = -Combined_Key, names_to = 'day', values_to = "cumulative_cases") %>%
  mutate(day = mdy(day), daily_cases = cumulative_cases - lag(cumulative_cases))
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_double(),
    ##   iso2 = col_character(),
    ##   iso3 = col_character(),
    ##   Admin2 = col_character(),
    ##   Province_State = col_character(),
    ##   Country_Region = col_character(),
    ##   Combined_Key = col_character()
    ## )

    ## See spec(...) for full column specifications.

``` r
data_in %>% ggplot(aes(y = cumulative_cases, x = day)) + geom_line() + 
  theme_bw() + ylab('Cumulative Cases') + xlab('') + 
  ggtitle('Cumulative Covid 19 Infections in Gallatin County, MT') + 
  labs(caption = 'Data scraped from the COVID-19 Dashboard by the Center for Systems Science and Engineering (CSEE) at JHU')
```

![](Scrape_MSU_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
data_in %>% ggplot(aes(y = daily_cases, x = day)) + geom_line() + 
  theme_bw() + ylab('Daily Positive Cases') + xlab('') + 
  ggtitle('Daily Diagnosed Covid 19 Infections in Gallatin County, MT') + 
  labs(caption = 'Data scraped from the COVID-19 Dashboard by the Center for Systems Science and Engineering (CSEE) at JHU')
```

![](Scrape_MSU_files/figure-gfm/unnamed-chunk-1-2.png)<!-- -->

### Enter Gallatin County Surveillance Data: Weekly MSU Counts

Note this needs to be updated on Friday. The current report format (PDF)
does not easily lend itself to data scraping.

``` r
Gallatin_Surveillance <- tibble(day = c('9/3/2020','9/10/2020', '9/17/2020', '9/24/2020', '10/2/2020', '10/8/2020', '10/15/2020',"10/22/2020", '10/29/2020','11/05/2020','11/12/2020'), 
                                new_MSU_cases = c(NA,3,7,66,43,60,65,99,132,212,265), 
                                cumulative_MSU_cases = c(38,41,48,114,157,217,282,381,513,725,990) ) %>%
                    mutate(day = mdy(day), period = as.character(1:length(new_MSU_cases))) %>% 
  full_join(data_in, by = 'day')%>% arrange(day)  %>%
  fill(period, .direction = 'up') %>%
  fill(new_MSU_cases, .direction = 'up') %>%
  fill(cumulative_MSU_cases, .direction = 'up')

## Filter county data prior to the first day of class

MSU <- Gallatin_Surveillance %>% 
  filter(day > mdy('08/16/2020')) %>% 
  dplyr::group_by(period) %>% 
  mutate(daily_proportion = daily_cases / sum(daily_cases)) %>% 
  ungroup() %>%
  mutate(daily_values = daily_proportion * new_MSU_cases, daily_integers = round(daily_values))


MSU %>% ggplot(aes(y = daily_integers, x = day)) + geom_line() + 
  theme_bw() + ylab('Daily Positive Cases') + xlab('') +ggtitle('Estimated Daily Diagnosed Covid 19 Infections for Montana State University') + 
  labs(caption = 'Data interpolated from data scraped from the  COVID-19 Dashboard by the Center for Systems Science \n and Engineering  at JHU and Gallatin County Weekly COVID-19 Surveillance Reports \n Monthly MSU cases are allocated to individual days using the proportion of Gallatin County cases on that day of the week')
```

    ## Warning: Removed 1 row(s) containing missing values (geom_path).

![](Scrape_MSU_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->