
# Explore FISHINTEL acoustic telemetry data

In this turorial we show a typical workflow for retreiving **acoustic telemetry data** from the ETN portal using the `etn` package. The `etn` package was developed by INBO, you can browse the source code at [https://github.com/inbo/etn/](https://github.com/inbo/etn/) or check the package's [page](https://github.com/inbo/etn/). This tutorial is based in Damiano Oldoni's article [Explore acoustic telemetry data](https://inbo.github.io/etn/articles/acoustic_telemetry.html) and the European Tracking Network's webinar on **["Management, visualization and analyses of aquatic telemetry data"](https://www.youtube.com/watch?v=f37cDY71iqM)**.

## Introduction

### Installation
You can install this package from GitHub with:

```
# install.packages("devtools")
devtools::install_github("inbo/etn")
```

### Load packages

Load the package with

```
library(etn)
```

We will also use the packages [dplyr](https://dplyr.tidyverse.org/) for data exploration, [lubridate](https://lubridate.tidyverse.org/) to handle datetime data and [leaflet](https://rstudio.github.io/leaflet/) for interactive map visualizations. 

Load these packages (install them if necessary)

```
# install packages
# install.packages("dplyr")
# install.packages("lubridate")
# install.packages("leaflet")

# load packages
library(dplyr)
library(lubridate)
library(leaflet)
```

Connect with your user credentials (as received by VLIZ) to the database. To not expose such confidential information, you could opt to use [environment variables](https://db.rstudio.com/best-practices/managing-credentials/#use-environment-variables):

``` {r}
con <- connect_to_etn(Sys.getenv("userid"), Sys.getenv("pwd"))
# This is the default, so you could also use connect_to_etn()

con <- connect_to_etn(username = "username", password = "password")
```

Using con as variable to store the collection is not mandatory, but it makes your life much easier as con is the default value of the argument connection, present in every other function of this package

## Select project or projects of interest

Not sure of what your project code is? Let's get an overview of all projects:

``` {r}
all_projects <- get_animal_projects()

# show a preview
all_projects %>% head(10)
```

```{r}
# A tibble: 10 × 11
#    project_id project_code    project_type telemetry_type project_name start_date end_date   latitude longitude #moratorium
#         <int> <chr>           <chr>        <chr>          <chr>        <date>     <date>        <dbl>     <dbl> <chr>
#  1        793 2004_Gudena     animal       Acoustic       2004_Gudena  2004-01-01 2005-12-31     56.4      9.91 1
#  2         16 2010_phd_reube… animal       Acoustic       2010_phd_re… 2010-08-01 2012-10-30     NA       NA    0
#  3        841 2010_phd_reube… animal       Acoustic       2010_phd_re… 2010-08-01 2012-10-30     NA       NA    0
#  4        760 2011_Loire      animal       Acoustic       2011_Loire   2011-11-11 2012-02-25     47.3     -1.98 1
#  5         20 2011_rivierprik animal       Acoustic       2011 Rivier… 2011-01-01 2012-09-03     NA       NA    0
#  6        754 2011_Warnow     animal       Acoustic       2011_Warnow  2011-06-01 2012-10-12     54.1     12.1  1
#  7         15 2012_leopoldka… animal       Acoustic       2012 Leopol… 2012-01-01 2016-01-18     NA       NA    0
#  8         18 2013_albertkan… animal       Acoustic       2013 Albert… 2013-10-10 2018-12-31     NA       NA    0
#  9        757 2013_Foyle      animal       Acoustic       2013_Foyle   2013-07-01 2014-03-01     54.9     -7.39 1
# 10         21 2014_demer      animal       Acoustic       2014 Demer   2014-04-10 2015-02-13     NA       NA    0
# # … with 1 more variable: imis_dataset_id <int>
```

If you know the `project code` already and you are just interested to get more information about it, you can specify it in the `get_animal_projects()` function directly:

``` {r}
fishintel_animal_project <- get_animal_projects(animal_project_code = "FISHINTEL")
fishintel_animal_project
```
```{r}
# # A tibble: 2 × 11
#   project_id project_code project_type telemetry_type project_name     start_date end_date   latitude longitude moratorium
#        <int> <chr>        <chr>        <chr>          <chr>            <date>     <date>        <dbl>     <dbl> <chr>
# 1        945 FISHINTEL    animal       Acoustic       Fisheries Innov… 2021-02-01 2023-06-30     50.2     -1.69 1
# 2        981 FISHINTEL    animal       ADST           Fisheries Innov… 2021-02-01 2023-06-30     NA       NA    1
# # … with 1 more variable: imis_dataset_id <int>
```

This is exactly the same as retrieving all projects first and filtering them afterwards based on column `project_code`:







