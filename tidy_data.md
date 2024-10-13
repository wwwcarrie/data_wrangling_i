tidy_data
================

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

## `pivot_longer`

Load the PULSE data

``` r
pulse_data = 
  haven ::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor :: clean_names()
```

wide format to long format ..

``` r
pulse_data_tidy = 
  pulse_data %>% 
  pivot_longer (
    bdi_score_bl:bdi_score_12m, 
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  )
```

rewrite, combine and extend (to add a mutate)

``` r
pulse_data = 
  haven ::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor :: clean_names() %>% 
  pivot_longer (
    bdi_score_bl:bdi_score_12m, 
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  relocate(id, visit) %>% 
  mutate(visit = recode(visit, "bl" = "00m"))
```

## `pivot_wider`

Make up some data!

``` r
analysis_result = 
  tibble (
    group = c("treatment", "treatment", "placebo", "placebo"),
    time = c("pre", "post","pre", "post"),
    mean = c(4, 8, 3.5, 4)
  )


analysis_result %>% 
  pivot_wider(
    names_from = "time",
    values_from = "mean"
  )
```

    ## # A tibble: 2 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

## binding rows

Using the LotR data.

First step: import each table.

``` r
fellowship_ring = 
  readxl :: read_excel ("./data/LotR_Words.xlsx", range = "B3:D6" ) %>% 
  mutate (movie = "fellowship_ring")

two_towers = 
  readxl :: read_excel ("./data/LotR_Words.xlsx", range = "F3:H6" ) %>% 
  mutate (movie = "two_towers")

return_king = 
  readxl :: read_excel ("./data/LotR_Words.xlsx", range = "J3:L6" ) %>% 
  mutate (movie = "return_king")
```

Bind all the rows together

``` r
lotr_tidy = 
  bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names () %>% 
  relocate(movie) %>% ## put "movie" into the first column
  pivot_longer (
    female:male, 
    names_to = "gender",
    values_to = "words"
  )
```

## joining datasets

Import the FAS datasets.

``` r
pups_df = 
  read_csv("./data/FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex, `1` = "male", `2` = "female")) ##change 1 to male, 2 to female
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Litter Number, PD ears
    ## dbl (4): Sex, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df = 
  read_csv("./data/FAS_litters.csv") %>% 
  janitor:: clean_names() %>% 
  relocate(litter_number) %>% ## "litter number" moved to the first column
  separate(group, into = c("dose", "day_of_tx"), sep = 3) ##separate up to 3 characters
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (4): Group, Litter Number, GD0 weight, GD18 weight
    ## dbl (4): GD of Birth, Pups born alive, Pups dead @ birth, Pups survive
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Next up, time to join them!

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number") %>% 
  arrange(litter_number) %>% 
  relocate(litter_number, dose, day_of_tx)
```
