
# Explore FISHINTEL acoustic telemetry data

In this turorial we show a typical workflow for retreiving **acoustic telemetry data** from the ETN portal using the `etn` package. The `etn` package was developed by INBO, you can browse the source code at [https://github.com/inbo/etn/](https://github.com/inbo/etn/) or check the package's [page](https://github.com/inbo/etn/). This tutorial is based in Damiano Oldoni's article [Explore acoustic telemetry data](https://inbo.github.io/etn/articles/acoustic_telemetry.html) and the European Tracking Network's webinar on **["Management, visualization and analyses of aquatic telemetry data"](https://www.youtube.com/watch?v=f37cDY71iqM)**.

### Topics on this tutorial
1. [Introduction](#introduction)
2. [Select a project of interest](#project)
3. [Animals](#animals)
4. [Detections](#detections)
5. [Stations](#stations)
6. [Acoustic tags](#tags)
7. [Acoustic network projects](#network)

## Introduction <a name="introduction"></a>

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

## Select a project of interest <a name="project"></a>

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

```{r}
all_projects %>%
  filter(project_code == "FISHINTEL")
```

To list all available animal project codes as a vector, you can use list_animal_project_codes(), one of the etn functions in the [`list_*` family](https://inbo.github.io/etn/reference/index.html#section-list-parameter-options):

```{r}
list_animal_project_codes() %>% head(10)

##  [1] "2004_Gudena"           "2010_phd_reubens"      "2010_phd_reubens_sync"
##  [4] "2011_Loire"            "2011_rivierprik"       "2011_Warnow"          
##  [7] "2012_leopoldkanaal"    "2013_albertkanaal"     "2013_Foyle"           
## [10] "2014_demer"
```


## Animals <a name="animals"></a>

By using `get_animals()` you can retrieve information about each animal (`animal_id`), such as scientific name, length, capture/release date and location, and the attached tag(s) (`tag_serial_number`):

```{r}
fishintel_animals <- get_animals(animal_project_code = "FISHINTEL")
fishintel_animals %>% head(5)
```

What species and how many individuals are tracked for the `FISHINTEL` project?

```
fishintel_animals %>% count(scientific_name)
```

## Detections <a name="detections"></a>

Let’s say we are interested in checking the data of bluefin tuna (*Thunnus thynnus*). You can retreive the detection history using `get_acoustic_detections`:

```
fishintel_detections_tuna <- get_acoustic_detections(
  animal_project_code = "FISHINTEL",
  scientific_name = "Thunnus thynnus"
)
```

Preview

```
fishintel_detections_tuna %>% head(5)
```
```
# # A tibble: 5 × 20
#   detection_id date_time           tag_serial_number acoustic_tag_id animal_project_code animal_id scientific_name
#          <int> <dttm>              <chr>             <chr>           <chr>                   <int> <chr>          
# 1    590293357 2021-10-28 15:15:39 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
# 2    590293358 2021-10-28 15:16:58 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
# 3    590293359 2021-10-28 15:17:48 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
# 4    590293360 2021-10-28 15:18:26 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
# 5    590293361 2021-10-28 15:19:53 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
# # … with 13 more variables: acoustic_project_code <chr>, receiver_id <chr>, station_name <chr>, deploy_latitude <dbl>,
# #   deploy_longitude <dbl>, sensor_value <dbl>, sensor_unit <chr>, sensor2_value <dbl>, sensor2_unit <chr>,
# #   signal_to_noise_ratio <int>, source_file <chr>, qc_flag <chr>, deployment_id <int>
```

You can also specify a time range, for example, check for detections within the month of september:

```
fishintel_september_detections_tuna <- get_acoustic_detections(
  animal_project_code = "FISHINTEL",
  start_date = "2021-10-01",
  end_date = "2021-10-31", # The end date is exclusive
  scientific_name = "Thunnus thynnus"
)
fishintel_september_detections_tuna
```
```
# # A tibble: 45 × 20
#    detection_id date_time           tag_serial_number acoustic_tag_id animal_project_code animal_id scientific_name
#           <int> <dttm>              <chr>             <chr>           <chr>                   <int> <chr>          
#  1    590293357 2021-10-28 15:15:39 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  2    590293358 2021-10-28 15:16:58 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  3    590293359 2021-10-28 15:17:48 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  4    590293360 2021-10-28 15:18:26 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  5    590293361 2021-10-28 15:19:53 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  6    590293362 2021-10-28 15:21:21 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  7    590293363 2021-10-28 15:22:35 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  8    590293364 2021-10-28 15:23:26 21272746          A69-1303-2746   FISHINTEL               10955 Thunnus thynnus
#  9    590293365 2021-10-28 17:09:44 21272748          A69-1303-2748   FISHINTEL               10962 Thunnus thynnus
# 10    590293366 2021-10-28 17:10:34 21272748          A69-1303-2748   FISHINTEL               10962 Thunnus thynnus
# # … with 35 more rows, and 13 more variables: acoustic_project_code <chr>, receiver_id <chr>, station_name <chr>,
# #   deploy_latitude <dbl>, deploy_longitude <dbl>, sensor_value <dbl>, sensor_unit <chr>, sensor2_value <dbl>,
# #   sensor2_unit <chr>, signal_to_noise_ratio <int>, source_file <chr>, qc_flag <chr>, deployment_id <int>
```

Which individuals have been detected (animal_id) and in which period?

```
fishintel_period_detections_tuna <-
  fishintel_detections_tuna %>%
  mutate(date = date(date_time)) %>%
  group_by(animal_id) %>%
  summarize(
    start = min(date),
    end = max(date)
  )
fishintel_period_detections_tuna
```
```
# # A tibble: 5 × 3
#   animal_id start      end       
#       <int> <date>     <date>    
# 1     10955 2021-10-28 2021-10-28
# 2     10958 2021-10-13 2021-10-13
# 3     10959 2021-10-28 2021-10-29
# 4     10962 2021-10-28 2021-10-28
# 5     10970 2021-09-15 2021-09-15
```

Notice we group by `animal_id`, the unique identifier of the fish. However, if the fish has only been tagged once (as typically occurs), we could use `acoustic_tag_id` as well, i.e. the identifier picked up by acoustic receivers:

```
fishintel_detections_tuna %>%
  mutate(date = date(date_time)) %>%
  group_by(acoustic_tag_id) %>%
  summarize(
    start = min(date),
    end = max(date)
  )
```
```
# # A tibble: 5 × 3
#   acoustic_tag_id start      end       
#   <chr>           <date>     <date>    
# 1 A69-1303-2746   2021-10-28 2021-10-28
# 2 A69-1303-2748   2021-10-28 2021-10-28
# 3 A69-1303-2749   2021-10-13 2021-10-13
# 4 A69-1303-2766   2021-10-28 2021-10-29
# 5 A69-1303-2769   2021-09-15 2021-09-15
```

We can also get the tracking duration of each fish:

```
fishintel_duration_detections_tuna <-
  fishintel_detections_tuna %>%
  group_by(animal_id) %>%
  summarize(duration = max(date_time) - min(date_time))
fishintel_duration_detections_tuna
```
```
# # A tibble: 5 × 2
#   animal_id duration  
#       <int> <drtn>    
# 1     10955   467 secs
# 2     10958  1200 secs
# 3     10959 67505 secs
# 4     10962 23433 secs
# 5     10970  2661 secs
```

How many times has an individual has been detected?

```
fishintel_detections_tuna %>%
  group_by(animal_id) %>%
  count()
```
```
# # A tibble: 5 × 2
# # Groups:   animal_id [5]
#   animal_id     n
#       <int> <int>
# 1     10955     8
# 2     10958    18
# 3     10959    15
# 4     10962     4
# 5     10970    36
```

## Stations <a name="stations"></a>

In how many statuins have the individuals been detected?

```
fishintel_detections_tuna %>%
  group_by(animal_id) %>%
  distinct(station_name) %>%
  count()
```
```
# A tibble: 5 × 2
# Groups:   animal_id [5]
  animal_id     n
      <int> <int>
1     10955     1
2     10958     1
3     10959     2
4     10962     2
5     10970     1
```

Which stations have been involved? You can retrieve them using [`list_values`](https://inbo.github.io/etn/reference/list_values.html) function applied to column `station_name`:

```
fishintel_stations_tuna <-
  fishintel_detections_tuna %>%
  list_values(station_name)
```
```
# 3 unique station_name values
```
```
fishintel_stations_tuna
# [1] "gwineas"  "falmouth" "dodman"  
```

Notice how a detection station can be linked to multiple deployments:

```
fishintel_detections_tuna %>%
  distinct(station_name, deployment_id) %>%
  group_by(station_name) %>%
  add_tally() %>%
  arrange(desc(n))
```
```
# # A tibble: 3 × 3
# # Groups:   station_name [3]
#   station_name deployment_id     n
#   <chr>                <int> <int>
# 1 gwineas              13138     1
# 2 falmouth             13136     1
# 3 dodman               13137     1
```

It’s also interesting to know the number of unique individuals per station:

```
fishintel_stations_tuna_n <-
  fishintel_detections_tuna %>%
  distinct(station_name, animal_id) %>%
  group_by(station_name) %>%
  count()
fishintel_stations_tuna_n
```
```
# # A tibble: 3 × 2
# # Groups:   station_name [3]
#   station_name     n
#   <chr>        <int>
# 1 dodman           1
# 2 falmouth         3
# 3 gwineas          3
```

## Acoustic tags <a name="tags"></a>

To get more information about the tags involved in tuna detections (`fishintel_detections_tuna`), you can use the function `get_tags`, which returns tag related information such as `serial number`, `manufacturer`, `model`, and `frequency`:

```
fishintel_tuna_tags_id <- list_values(fishintel_detections_tuna, acoustic_tag_id)
```
```
# 5 unique acoustic_tag_id values
```

```
fishintel_tuna_tags_id

#[1] "A69-1303-2746" "A69-1303-2748" "A69-1303-2749" "A69-1303-2766" "A69-1303-2769"
```

You can also retrieve such information by tag_serial_number:

```
fishintel_tuna_tags_serial <- unique(fishintel_detections_tuna$tag_serial_number)
tags_tuna_serial <- get_tags(tag_serial_number = fishintel_tuna_tags_serial)
tags_tuna_serial
```
```
# # A tibble: 5 × 54
#   tag_serial_number tag_type tag_subtype sensor_type acoustic_tag_id acoustic_tag_id_alternative manufacturer  model frequency status activation_date
#   <chr>             <chr>    <chr>       <chr>       <chr>           <chr>                       <chr>         <chr> <chr>     <chr>  <dttm>
# 1 21272746          acoustic animal      NA          R64K-2746       A69-1303-2746               THELMA BIOTEL HP-16 069K      active 2021-08-27 15:28:00
# 2 21272748          acoustic animal      NA          R64K-2748       A69-1303-2748               THELMA BIOTEL HP-16 069K      active 2021-08-29 17:54:00
# 3 21272749          acoustic animal      NA          R64K-2749       A69-1303-2749               THELMA BIOTEL HP-16 069K      active 2021-08-28 11:12:00
# 4 21272766          acoustic animal      NA          R64K-2766       A69-1303-2766               THELMA BIOTEL HP-16 069K      active 2021-08-29 09:41:00
# 5 21272769          acoustic animal      NA          R64K-2769       A69-1303-2769               THELMA BIOTEL HP-16 069K      active 2021-09-14 15:33:00
# # … with 43 more variables:
```

> :warning: However, keep in mind that there is a fundamental difference between the arguments `acoustic_tag_id` and `tag_serial_number`: the `tag_serial_number` identifies the device, which could contain multiple tags or sensors and thus multiple `acoustic_tag_id`.


##  Acoustic network projects <a name="network"></a>

The detection of tuna for the FISHITNTEL project is possible due to an array of receivers and deployments, that in ETN constitutes a network project. These are mentioned in the field `acoustic_project_code`. You can retreive them via de *list* function `list_values()`:

```
fishintel_network_project <- fishintel_detections_tuna %>%
  list_values(acoustic_project_code)
```
```
# 1 unique acoustic_project_code values
```
```
fishintel_network_project
```
```
# [1] "FISHINTEL"
```

To get more information about this network project, you can use function `get_acoustic_projects()`

```
fishintel_network_project_info <- get_acoustic_projects(
  acoustic_project_code = fishintel_network_project
)
fishintel_network_project_info
```
```
# # A tibble: 1 × 11
#   project_id project_code project_type telemetry_type project_name                        start_date end_date   latitude longitude moratorium imis_dataset_id
#        <int> <chr>        <chr>        <chr>          <chr>                               <date>     <date>        <dbl>     <dbl> <chr>                <int>
# 1        946 FISHINTEL    acoustic     Acoustic       Fisheries Innovation for sustainab… 2021-02-01 2023-06-30     50.2     -1.69 0                     7881
```














