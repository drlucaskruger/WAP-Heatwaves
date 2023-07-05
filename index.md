---
title: "Detection of extreme heatwaves and low sea ice cover in the Antarctic Peninsula
 for evaluation of effects over Krill"
subtitle: "Sumplementary document"
author: "Lucas Krüger, Maurício Mardones, Lorena Rebolledo"
date:  "05 July, 2023"
bibliography: heatwave.bib
csl: apa.csl
link-citations: yes
linkcolor: blue
output:
  html_document:
    keep_md: true
    toc: true
    toc_deep: 3
    toc_float:
      collapsed: false
      smooth_scroll: false
    theme: cosmo
    fontsize: 0.9em
    linestretch: 1.7
    html-math-method: katex
    self-contained: true
    code-tools: true
editor_options: 
  markdown: 
    wrap: 72
---



## Background

We use [heattwavesR package](https://robwschlegel.github.io/heatwaveR/index.html) to extract period and grid relative with our porpouses [@heatwavesR]. This package use @reraddp2023 source `reraddp` package describe in [this repo](https://github.com/ropensci/rerddap) and with this [documentation](https://cran.r-project.org/web/packages/rerddap/rerddap.pdf).


## Load packages


```r
#remotes::install_github("ropensci/rerddap")

library(dplyr) # A staple for modern data management in R
library(lubridate) # Useful functions for dealing with dates
library(ggplot2) # The preferred library for data visualisation
library(tidync) # For easily dealing with NetCDF data
library(rerddap) # For easily downloading subsets of data
library(doParallel) # For parallel processing
library(heatwaveR) # heatwaves analysis
library(udunits2) # spatial units library
library(patchwork) # for joining plots
library(raster) #sp is going to retire
library(terra) # for spatial objects
library(sf)# for spatial objects
```

## Set plot themes (panel spacing will be useful)



```r
th<- theme(axis.text=element_text(size=12, face="bold",colour="grey30"),
           axis.title=element_text(size=12,face="bold"),
           legend.text = element_text(size=12),
           panel.grid.major = element_blank(),
           panel.grid.minor = element_blank(),
           title =element_text(size=12, face="bold",colour="black"),
           panel.spacing = unit(1, "lines")) # theme for plots
```

## Download data


```r
#---------- Download NOAA erdap optimum interpolation SST and SIC----
# The information for the NOAA OISST data
#rerddap::info(datasetid = "ncdcOisst21Agg_LonPM180", url = "https://coastwatch.pfeg.noaa.gov/erddap/")

OISST_sub_dl <- function(time_df){
  OISST_dat <- griddap(x = "ncdcOisst21Agg_LonPM180", 
                       url = "https://coastwatch.pfeg.noaa.gov/erddap/", 
                       time = c(time_df$start, time_df$end), 
                       zlev = c(0, 0),
                       latitude = c(-76, -55),
                       longitude = c(-90, -30),
                       fields = c("sst","ice"))$data %>% 
    mutate(time = as.Date(stringr::str_remove(time, "T00:00:00Z"))) %>% 
    dplyr::rename(t = time, temp = sst) %>% 
    select(lon, lat, t, temp,ice) 
}

# 30 years of data to calculate heatwaves: data will have to be downloaded each 5 years
dl_years1 <- data.frame(date_index = 1:6,
            start = as.Date(c("1995-01-01", "2000-01-01", "2006-01-01", "2013-01-01","2019-01-01","2021-01-01")),
            end = as.Date(c("1999-12-31", "2005-12-31", "2012-12-31", "2018-12-31","2020-12-31","2022-12-31")))

dl_years2<-data.frame(date_index = 1:2,
                      start = as.Date(c("2018-01-01","2021-01-01")),
                      end = as.Date(c("2020-12-31","2022-06-30")))
```


Download all of the data with one nested request. 
The time this takes depends on connection speed download on two separated commands was the solution to the download not to crash



```r
system.time(
  OISST_data <- dl_years2 %>% 
    group_by(date_index) %>% 
    group_modify(~OISST_sub_dl(.x)) %>% 
    ungroup() %>% 
    dplyr::select(lon, lat, t, temp,ice)
) 

system.time(
  OISST_data2 <- dl_years2 %>% 
    group_by(date_index) %>% 
    group_modify(~OISST_sub_dl(.x)) %>% 
    ungroup() %>% 
    dplyr::select(lon, lat, t, temp,ice)
) 
```

## Data handling

## Plot data

## References
