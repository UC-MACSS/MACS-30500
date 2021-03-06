---
title: "Practice drawing vector maps"
date: 2019-03-01

type: docs
toc: true
draft: false
categories: ["dataviz", "geospatial"]

menu:
  notes:
    parent: Geospatial visualization
    weight: 4
---




```r
library(tidyverse)
library(sf)
library(tidycensus)
library(viridis)

# useful on MacOS to speed up rendering of geom_sf() objects
if (!identical(getOption("bitmapType"), "cairo") && isTRUE(capabilities()[["cairo"]])) {
  options(bitmapType = "cairo")
}

options(digits = 3)
set.seed(1234)
theme_set(theme_minimal())
```

## American Community Survey

The U.S. Census Bureau conducts the [American Community Survey](https://www.census.gov/programs-surveys/acs) which gathers detailed information on topics such as demographics, employment, educational attainment, etc. They make a vast portion of their data available through an [application programming interface (API)](/notes/application-program-interface/), which can be accessed intuitively through R via the [`tidycensus` package](https://walkerke.github.io/tidycensus/index.html). We previously discussed how to use this package to [obtain statistical data from the decennial census](/notes/application-program-interface/#census-data-with-tidycensus). However the Census Bureau also has detailed information on political and geographic boundaries which we can combine with their statistical measures to easily construct geospatial visualizations.

{{% callout note %}}

If you have not already, [obtain an API key](https://api.census.gov/data/key_signup.html) and [store it securely](/notes/application-program-interface/#census-data-with-tidycensus) on your computer.

{{% /callout %}}

## Exercise: Visualize income data

1. Obtain information on median household income in 2017 for Cook County, IL at the tract-level using the ACS. To retrieve the geographic features for each tract, set `geometry = TRUE` in your function.

    {{% callout note %}}
    
    You can use `load_variables(year = 2017, dataset = "acs5")` to retrieve the list of variables available and search to find the correct variable name.
    
    {{% /callout %}}

    {{< spoiler text="Click for the solution" >}}
    

```r
cook_inc <- get_acs(
  state = "IL",
  county = "Cook",
  geography = "tract",
  variables = c(medincome = "B19013_001"),
  year = 2017,
  geometry = TRUE
)
```


```r
cook_inc
```

```
## Simple feature collection with 1319 features and 5 fields (with 1 geometry empty)
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -88.3 ymin: 41.5 xmax: -87.5 ymax: 42.2
## Geodetic CRS:  NAD83
## First 10 features:
##          GEOID                                       NAME  variable estimate
## 1  17031010201 Census Tract 102.01, Cook County, Illinois medincome    40841
## 2  17031030200    Census Tract 302, Cook County, Illinois medincome    64089
## 3  17031031700    Census Tract 317, Cook County, Illinois medincome    44555
## 4  17031031900    Census Tract 319, Cook County, Illinois medincome    61211
## 5  17031050200    Census Tract 502, Cook County, Illinois medincome    74375
## 6  17031051300    Census Tract 513, Cook County, Illinois medincome   149271
## 7  17031061500    Census Tract 615, Cook County, Illinois medincome   117656
## 8  17031062600    Census Tract 626, Cook County, Illinois medincome   144211
## 9  17031063400    Census Tract 634, Cook County, Illinois medincome    95488
## 10 17031070600    Census Tract 706, Cook County, Illinois medincome   151250
##      moe                       geometry
## 1   7069 MULTIPOLYGON (((-87.7 42, -...
## 2  12931 MULTIPOLYGON (((-87.7 42, -...
## 3  12220 MULTIPOLYGON (((-87.7 42, -...
## 4   6343 MULTIPOLYGON (((-87.7 42, -...
## 5  18773 MULTIPOLYGON (((-87.7 42, -...
## 6  26389 MULTIPOLYGON (((-87.7 41.9,...
## 7  11416 MULTIPOLYGON (((-87.7 41.9,...
## 8  22537 MULTIPOLYGON (((-87.7 41.9,...
## 9   4904 MULTIPOLYGON (((-87.6 41.9,...
## 10 47800 MULTIPOLYGON (((-87.7 41.9,...
```
    
    {{< /spoiler >}}

1. Draw a choropleth using the median household income data. Use a continuous color gradient to identify each tract's median household income.

    {{< spoiler text="Click for the solution" >}}


```r
ggplot(data = cook_inc) +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = estimate, color = estimate)) +
  # increase interpretability of graph
  scale_color_continuous(labels = scales::dollar) +
  scale_fill_continuous(labels = scales::dollar) +
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/income-cook-map-1.png" width="672" />

    {{< /spoiler >}}

## Exercise: Customize your maps

1. Draw the same choropleth for Cook County, but convert median household income into a discrete variable with 6 levels.

    {{< spoiler text="Click for the solution" >}}

    * Using `cut_interval()`:


```r
cook_inc %>%
  mutate(inc_cut = cut_interval(estimate, n = 6)) %>%
  ggplot() +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = inc_cut, color = inc_cut)) +
  # increase interpretability of graph
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cut-interval-1.png" width="672" />
        
    * Using `cut_number()`:


```r
cook_inc %>%
  mutate(inc_cut = cut_number(estimate, n = 6)) %>%
  ggplot() +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = inc_cut, color = inc_cut)) +
  # increase interpretability of graph
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cut-number-1.png" width="672" />

    {{< /spoiler >}}

1. Draw the same choropleth for Cook County using the discrete variable, but select an appropriate color palette using [Color Brewer](/notes/optimal-color-palettes/#color-brewer).

    {{< spoiler text="Click for the solution" >}}

    * Using `cut_interval()` and the Blue-Green palette:
    

```r
cook_inc %>%
  mutate(inc_cut = cut_interval(estimate, n = 6)) %>%
  ggplot() +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = inc_cut, color = inc_cut)) +
  scale_fill_brewer(type = "seq", palette = "BuGn") +
  scale_color_brewer(type = "seq", palette = "BuGn") +
  # increase interpretability of graph
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cut-interval-optimal-1.png" width="672" />
        
    * Using `cut_number()` and the Blue-Green palette:


```r
cook_inc %>%
  mutate(inc_cut = cut_number(estimate, n = 6)) %>%
  ggplot() +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = inc_cut, color = inc_cut)) +
  scale_fill_brewer(type = "seq", palette = "BuGn") +
  scale_color_brewer(type = "seq", palette = "BuGn") +
  # increase interpretability of graph
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cut-number-optimal-1.png" width="672" />
        
    You can choose any palette that is for sequential data.
    
    {{< /spoiler >}}

1. Use the [`viridis` color palette](/notes/optimal-color-palettes/#viridis) for the Cook County map drawn using the continuous measure.

    {{< spoiler text="Click for the solution" >}}


```r
ggplot(data = cook_inc) +
  # use fill and color to avoid gray boundary lines
  geom_sf(aes(fill = estimate, color = estimate)) +
  # increase interpretability of graph
  scale_color_viridis(labels = scales::dollar) +
  scale_fill_viridis(labels = scales::dollar) +
  labs(
    title = "Median household income in Cook County, IL",
    subtitle = "In 2017",
    color = NULL,
    fill = NULL,
    caption = "Source: American Community Survey"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/income-cook-map-viridis-1.png" width="672" />

    {{< /spoiler >}}

### Session Info



```r
devtools::session_info()
```

```
## ─ Session info ───────────────────────────────────────────────────────────────
##  setting  value                       
##  version  R version 4.0.4 (2021-02-15)
##  os       macOS Big Sur 10.16         
##  system   x86_64, darwin17.0          
##  ui       X11                         
##  language (EN)                        
##  collate  en_US.UTF-8                 
##  ctype    en_US.UTF-8                 
##  tz       America/Chicago             
##  date     2021-05-25                  
## 
## ─ Packages ───────────────────────────────────────────────────────────────────
##  package     * version date       lib source        
##  assertthat    0.2.1   2019-03-21 [1] CRAN (R 4.0.0)
##  backports     1.2.1   2020-12-09 [1] CRAN (R 4.0.2)
##  blogdown      1.3     2021-04-14 [1] CRAN (R 4.0.2)
##  bookdown      0.22    2021-04-22 [1] CRAN (R 4.0.2)
##  broom         0.7.6   2021-04-05 [1] CRAN (R 4.0.4)
##  bslib         0.2.5   2021-05-12 [1] CRAN (R 4.0.4)
##  cachem        1.0.5   2021-05-15 [1] CRAN (R 4.0.2)
##  callr         3.7.0   2021-04-20 [1] CRAN (R 4.0.2)
##  cellranger    1.1.0   2016-07-27 [1] CRAN (R 4.0.0)
##  class         7.3-19  2021-05-03 [1] CRAN (R 4.0.2)
##  classInt      0.4-3   2020-04-07 [1] CRAN (R 4.0.0)
##  cli           2.5.0   2021-04-26 [1] CRAN (R 4.0.2)
##  colorspace    2.0-1   2021-05-04 [1] CRAN (R 4.0.2)
##  crayon        1.4.1   2021-02-08 [1] CRAN (R 4.0.2)
##  DBI           1.1.1   2021-01-15 [1] CRAN (R 4.0.2)
##  dbplyr        2.1.1   2021-04-06 [1] CRAN (R 4.0.4)
##  desc          1.3.0   2021-03-05 [1] CRAN (R 4.0.2)
##  devtools      2.4.1   2021-05-05 [1] CRAN (R 4.0.2)
##  digest        0.6.27  2020-10-24 [1] CRAN (R 4.0.2)
##  dplyr       * 1.0.6   2021-05-05 [1] CRAN (R 4.0.2)
##  e1071         1.7-6   2021-03-18 [1] CRAN (R 4.0.2)
##  ellipsis      0.3.2   2021-04-29 [1] CRAN (R 4.0.2)
##  evaluate      0.14    2019-05-28 [1] CRAN (R 4.0.0)
##  fansi         0.4.2   2021-01-15 [1] CRAN (R 4.0.2)
##  fastmap       1.1.0   2021-01-25 [1] CRAN (R 4.0.2)
##  forcats     * 0.5.1   2021-01-27 [1] CRAN (R 4.0.2)
##  foreign       0.8-81  2020-12-22 [1] CRAN (R 4.0.4)
##  fs            1.5.0   2020-07-31 [1] CRAN (R 4.0.2)
##  generics      0.1.0   2020-10-31 [1] CRAN (R 4.0.2)
##  ggplot2     * 3.3.3   2020-12-30 [1] CRAN (R 4.0.2)
##  glue          1.4.2   2020-08-27 [1] CRAN (R 4.0.2)
##  gridExtra     2.3     2017-09-09 [1] CRAN (R 4.0.0)
##  gtable        0.3.0   2019-03-25 [1] CRAN (R 4.0.0)
##  haven         2.4.1   2021-04-23 [1] CRAN (R 4.0.2)
##  here          1.0.1   2020-12-13 [1] CRAN (R 4.0.2)
##  hms           1.1.0   2021-05-17 [1] CRAN (R 4.0.4)
##  htmltools     0.5.1.1 2021-01-22 [1] CRAN (R 4.0.2)
##  httr          1.4.2   2020-07-20 [1] CRAN (R 4.0.2)
##  jquerylib     0.1.4   2021-04-26 [1] CRAN (R 4.0.2)
##  jsonlite      1.7.2   2020-12-09 [1] CRAN (R 4.0.2)
##  KernSmooth    2.23-20 2021-05-03 [1] CRAN (R 4.0.2)
##  knitr         1.33    2021-04-24 [1] CRAN (R 4.0.2)
##  lattice       0.20-44 2021-05-02 [1] CRAN (R 4.0.2)
##  lifecycle     1.0.0   2021-02-15 [1] CRAN (R 4.0.2)
##  lubridate     1.7.10  2021-02-26 [1] CRAN (R 4.0.2)
##  magrittr      2.0.1   2020-11-17 [1] CRAN (R 4.0.2)
##  maptools      1.1-1   2021-03-15 [1] CRAN (R 4.0.2)
##  memoise       2.0.0   2021-01-26 [1] CRAN (R 4.0.2)
##  modelr        0.1.8   2020-05-19 [1] CRAN (R 4.0.0)
##  munsell       0.5.0   2018-06-12 [1] CRAN (R 4.0.0)
##  pillar        1.6.1   2021-05-16 [1] CRAN (R 4.0.4)
##  pkgbuild      1.2.0   2020-12-15 [1] CRAN (R 4.0.2)
##  pkgconfig     2.0.3   2019-09-22 [1] CRAN (R 4.0.0)
##  pkgload       1.2.1   2021-04-06 [1] CRAN (R 4.0.2)
##  prettyunits   1.1.1   2020-01-24 [1] CRAN (R 4.0.0)
##  processx      3.5.2   2021-04-30 [1] CRAN (R 4.0.2)
##  proxy         0.4-25  2021-03-05 [1] CRAN (R 4.0.2)
##  ps            1.6.0   2021-02-28 [1] CRAN (R 4.0.2)
##  purrr       * 0.3.4   2020-04-17 [1] CRAN (R 4.0.0)
##  R6            2.5.0   2020-10-28 [1] CRAN (R 4.0.2)
##  rappdirs      0.3.3   2021-01-31 [1] CRAN (R 4.0.2)
##  Rcpp          1.0.6   2021-01-15 [1] CRAN (R 4.0.2)
##  readr       * 1.4.0   2020-10-05 [1] CRAN (R 4.0.2)
##  readxl        1.3.1   2019-03-13 [1] CRAN (R 4.0.0)
##  remotes       2.3.0   2021-04-01 [1] CRAN (R 4.0.2)
##  reprex        2.0.0   2021-04-02 [1] CRAN (R 4.0.2)
##  rgdal         1.5-23  2021-02-03 [1] CRAN (R 4.0.2)
##  rlang         0.4.11  2021-04-30 [1] CRAN (R 4.0.2)
##  rmarkdown     2.8     2021-05-07 [1] CRAN (R 4.0.2)
##  rprojroot     2.0.2   2020-11-15 [1] CRAN (R 4.0.2)
##  rstudioapi    0.13    2020-11-12 [1] CRAN (R 4.0.2)
##  rvest         1.0.0   2021-03-09 [1] CRAN (R 4.0.2)
##  sass          0.4.0   2021-05-12 [1] CRAN (R 4.0.2)
##  scales        1.1.1   2020-05-11 [1] CRAN (R 4.0.0)
##  sessioninfo   1.1.1   2018-11-05 [1] CRAN (R 4.0.0)
##  sf          * 0.9-8   2021-03-17 [1] CRAN (R 4.0.2)
##  sp            1.4-5   2021-01-10 [1] CRAN (R 4.0.2)
##  stringi       1.6.1   2021-05-10 [1] CRAN (R 4.0.2)
##  stringr     * 1.4.0   2019-02-10 [1] CRAN (R 4.0.0)
##  testthat      3.0.2   2021-02-14 [1] CRAN (R 4.0.2)
##  tibble      * 3.1.1   2021-04-18 [1] CRAN (R 4.0.2)
##  tidycensus  * 0.11.4  2021-01-20 [1] CRAN (R 4.0.2)
##  tidyr       * 1.1.3   2021-03-03 [1] CRAN (R 4.0.2)
##  tidyselect    1.1.1   2021-04-30 [1] CRAN (R 4.0.2)
##  tidyverse   * 1.3.1   2021-04-15 [1] CRAN (R 4.0.2)
##  tigris        1.4     2021-05-16 [1] CRAN (R 4.0.4)
##  units         0.7-1   2021-03-16 [1] CRAN (R 4.0.2)
##  usethis       2.0.1   2021-02-10 [1] CRAN (R 4.0.2)
##  utf8          1.2.1   2021-03-12 [1] CRAN (R 4.0.2)
##  uuid          0.1-4   2020-02-26 [1] CRAN (R 4.0.0)
##  vctrs         0.3.8   2021-04-29 [1] CRAN (R 4.0.2)
##  viridis     * 0.6.1   2021-05-11 [1] CRAN (R 4.0.2)
##  viridisLite * 0.4.0   2021-04-13 [1] CRAN (R 4.0.2)
##  withr         2.4.2   2021-04-18 [1] CRAN (R 4.0.2)
##  xfun          0.23    2021-05-15 [1] CRAN (R 4.0.2)
##  xml2          1.3.2   2020-04-23 [1] CRAN (R 4.0.0)
##  yaml          2.2.1   2020-02-01 [1] CRAN (R 4.0.0)
## 
## [1] /Library/Frameworks/R.framework/Versions/4.0/Resources/library
```
