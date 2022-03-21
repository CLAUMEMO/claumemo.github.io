
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

```
con <- connect_to_etn(Sys.getenv("userid"), Sys.getenv("pwd"))
# This is the default, so you could also use connect_to_etn()

con <- connect_to_etn(username = "username", password = "password")
```

Using con as variable to store the collection is not mandatory, but it makes your life much easier as con is the default value of the argument connection, present in every other function of this package

## Select project or projects of interest



