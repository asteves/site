---
title: Plotting Democracy over Time with R
author: alex
date: '2020-07-25'
slug: plotting-democracy-over-time-with-r
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2020-07-25T10:38:37-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

## Introduction 

Recently, I made some maps for a research article. I initially had some reticence, as it had been a long time since I worked with GIS systems. To my (pleasant) surprise, the R spatial ecosystem has evolved to make the process extremely user friendly. In the spirit of "write it down to not forget," this post provides a beginning to end tutorial for plotting maps across time. To give the tutorial a practical application, I focus on plotting the electoral democracy changes across time using the V-Dem index. 

## What Packages do I need?

To follow along, you'll need the following R packages. I strongly recommend using RStudio for R projects as well. 


```r
install.packages(c("cshapes", "countrycode","here",
                   "rnaturalearth","rnaturalearthdata",
                   "rcartocolor","sf", "tidyverse"))
```


```r
library(cshapes)
library(countrycode)
library(data.table)
library(here)
library(rnaturalearth)
library(rnaturalearthdata)
library(rcartocolor)
library(sf)
library(tidyverse)
```

Here is what we're using each package for: 

- `cshapes` is an R package for the [cshapes](http://nils.weidmann.ws/projects/cshapes.html) project, a dataset that provides historical maps of state boundaries and capitals in the post-WW2 period. Since countries achieve independence at different times, we want access to shapefiles that take into account the world's political topography.  
- `countrycode` is my favorite utility package for doing comparative and IR coding work. Unquestionably, the most annoying aspect of IR datasets is the lack of standardization of country names. Is it United States? USA? United States of America? `countrycode` can convert to and from 600+ variants of country names.
- `here` is an excellent package for letting R take care of relative file paths. The package is a lifesaver for reproducibility. 
- `rnaturalearth` and `rnaturalearthdata` give us the current world map shapefiles. 
- `sf` is one of the workhorse GIS packages in R. It implements functions for objects that implement the simple feature access standard. 
- `rcartocolor` gives us pretty colors. As a colorblind individual, I am a big fan of packages that provide me access to pre-approved beautiful color palettes. 
- `tidyverse` for reading, munging, cleaning, and plotting our polygons and data. 

## Bring on the Polygons 

We will track electoral democracy following the end of the Cold War with three snapshots of the world: 1992, 2010, and 2019. Of course, to paraphrase Carl Sagan, if we wish to show democratic changes from scratch, we must first invent the universe. Here that means we need to read in the appropriate shapefiles to our environment. We will do that using functions from `cshapes` and `sf.` 

Note that `cshapes` has not been updated for maps after 2016, so we will use a different shapefile for the present day. You may receive a warning about a deprecated function, but that will not affect the code. 


```r
# cshp() requires we specify the date for our map 
# st_as_sf() converts the polygons to an sf object

## 1992 
map_92 <- cshapes::cshp(date = as.Date("1992-1-1"))%>%
  st_as_sf()

## 2010 
map_10 <- cshapes::cshp(date = as.Date("2010-1-1"))%>%
  st_as_sf()

## Present Day 
map_19 <- ne_countries(scale = "medium", type = "countries", returnclass = "sf")%>%
  # No one cares about Antartica or Greenland 
  # Well maybe Denmark cares about Greenland, but we don't
  filter(!region_wb == "Antarctica",
         !admin == "Greenland")%>%
  # We are going to join by Correlates of War Code 
  # This is given in cshapes data but not in ne_countries()
  # add COW Code using countrycode() and manually assign Serbia's
    mutate(sovereignt = str_replace_all(sovereignt,
                                        "Republic of Serbia", 
                                        "Serbia"),
        COWCODE = countrycode(sovereignt, 
                              "country.name", 
                              "cown"),
        COWCODE = ifelse(sovereignt == "Serbia", 
                         345, 
                         COWCODE))
```

After running this code block, we now have three shapefiles in our environment. 

## Join Democracy Data with Shapefile Data 

Our next step is to add data about electoral democracy to our shapefiles. Shapefiles have data attached, and we can see them in R by running `shp@data` where "shp" is the name of our shapefile. 

Our data on democracy comes from the [V-Dem](https://www.v-dem.net) project. Specifically, we are going to focus on the variable `v2x_polyarchy` which measures to what extent the ideal of electoral democracy is in its fullest sense achieved. Ignore the thorny political theory questions of what any of those words mean individually, much less together in a sentence. This variable is an index formed by taking a weighted average of five other indices measuring aspects of electoral freedom and a five-way multiplicative interaction between those indices. Again, best to ignore the measurement challenges. 

Presuming you have downloaded the V-Dem data, we can read it into R with `read_csv()`. The next code block presupposes that you have the data in R and named it vdem. 


```r
## 1992
vdem92 <- vdem %>% 
  filter(year == 1992)

## 2010 
vdem10 <- vdem %>% 
  filter(year == 2010)

## 2019
vdem19 <- vdem %>% 
  filter(year == 2019)
```

Now, we join these snapshots with our shapefile data. 


```r
# V-Dem doesn't cover some microstates, 
# We remove those with a filter 
map_1992 <- left_join(map_92, vdem92, 
                      by=c("COWCODE"="COWcode"))%>%
  filter(!is.na(v2x_polyarchy))
map_2010 <- left_join(map_10, vdem10, 
                      by=c("COWCODE"="COWcode"))%>%
  filter(!is.na(v2x_polyarchy))
map_2019 <- left_join(map_19, vdem19, 
                      by=c("COWCODE"="COWcode"))%>%
  filter(!is.na(v2x_polyarchy))
```

## Plot Pretty Maps

To not write the same ggplot calls three times, I just put them all in one function. 


```r
makeMap <- function(df, year = "Default"){
    g <- ggplot(df)+ 
      geom_sf(aes(fill = v2x_polyarchy), size = .1)+
      scale_fill_carto_c(name = "Democracy Level", palette="Purp")+
      coord_sf()+
      theme_void()+
        theme(legend.position = "bottom")+
        ggtitle(year)
    return(g)
}
```

Apply it to our data frames for pretty map wonder. 


```r
makeMap(map_1992, "1992")
makeMap(map_2010, "2010")
makeMap(map_2019, "2019")
```

![Democray Map 1992](/post/2020-07-25-plotting-democracy-over-time-with-r.en_files/map92.png)
![Democray Map 2010](/post/2020-07-25-plotting-democracy-over-time-with-r.en_files/map2010.png)
![Democracy Map 2019](/post/2020-07-25-plotting-democracy-over-time-with-r.en_files/map2019.png)

## Conclusion 

R's GIS ecosystem has evolved to be robust and user-friendly. This post scratches the surface of R's mapping capabilities. Readers should consult [RSpatial](https://rspatial.org/), an extensive list of spatial data analysis resources, and modeling resources. a
