---
title: 'Bulk downloads of pdfs from a website: An Example script'
author: alex
date: '2021-02-03'
slug: []
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2021-02-03T15:09:09-08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

I am working at [UC Berkeley's D-Lab](https://dlab.berkeley.edu/) as a Data Science Fellow. One of my responsibilities is to provide consulting to the UC Berkeley community on statistical and data science projects. A common request of late due to **points at everything** is to help with web scraping for projects. 

Recently, a request came in to scrape a page and download the pdf files that were linked. Fortunately, the page was simple from an HTML perspective, and I could apply a few common patterns to pull the downloads. Over the break, I read about a few productivity systems, all of which suggested writing notes to your future self. In that spirit, here's the current way I solve this kind of problem as an example script. 

I make use of the `purrr` for clean, functional programming, `rvest` for scraping, and `stringr` because I suck at regular expressions. 


```r
library(purrr)
library(rvest)
library(stringr)

downloadIt <- function(link){
  article <- str_extract(link, pattern = "article=[:digit:]+")
  out <- paste0(output_dir,"/",article , ".pdf")
  download.file(url = link, destfile = out, mode = "wb")
})

safelyDownloadIt <- safely(downloadIt)

# Output folder name 
output_dir <- "~/Desktop/DownloadedDocs"

# Sometimes it's helpful to make a specific directory on the fly
# This code will only create the directory if it does not
# currently exist 
if(!dir.exists(output_dir)){
  dir.create(output_dir)
  }

# Website of interest 
url <- "https://scholarworks.utep.edu/border_region/"

# Now we just chain away to bulk download all the pdfs 
# that exist on the page. 

url %>% 
  read_html()%>%
  html_nodes("a")%>%
  html_attr("href")%>%
  str_subset("viewcontent.cgi")%>%
  # Because our download is silent 
  # Use walk() instead of map()
  walk(safelyDownloadIt)
```


Clean, simple, and surprisingly quick. Of course, if one were planning to download many documents, I would suggest implementing some sleep function to play nicely with the server.
