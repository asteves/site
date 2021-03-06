---
title: Using pdftools to Extract Vote Data
author: alex
date: '2019-06-02'
slug: using-pdftools-to-extract-vote-data
categories:
  - R
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2019-06-02T16:43:30-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Introduction
------------

Suppose you are a concerned citizen who would like to know how voters in a state voted. Perhaps you are a voter in a state with [rampant corruption](https://www.vice.com/en_us/article/vba4vx/corruption-is-rampant-and-new-jersey-is-cool-with-it) or perhaps you are a voter in a state that does not have [paper backups for voting machines](https://www.app.com/story/news/politics/elections/2018/10/22/new-jersey-voting-machines-security/1656558002/). Perhaps you are just masochistic enough to be interested in pulling tables out of reasonably well formed pdfs. The following is a code example for the last one. 

Fortunately (or unfortunately) for us, the state of New Jersey still provides their official election results in pdf files instead of a common data format like csv or even an Excel file. Copying each item by hand risks user error through fat fingers, sheer tedium, or great displeasure at a state in the 21st Century still outputting results to pdf. Therefore, it is much preferable to search for a code solution. 

Each county in New Jersey reports Senate results separately. Here's Atlantic County's [precinct results](https://preview.tinyurl.com/y3zumyjk) for the 2018 Senate Race. The table that we are going to extract looks like [this](https://imgur.com/KEFhzjv). 

Here's one way to pull out the data, using some knowledge about who ran. I make use of tidyverse functions and pdftools. Each step of the function is commented. 


```r
library(tidyverse)
library(pdftools)
```

Once those are loaded, we can write our function. 


```r
# Remove columns that are all NAs
not_all_na <- function(x){
    !all(is.na(x))
}

senate_clean_table <- function(tbl){
    # Remove commas and split on new lines 
    tables <- NULL
    tbl <- str_replace_all(tbl,pattern = ",", "")%>%
        # We can see the table ends with the string NJDOE
        str_replace_all("Total[:print:]+", "NJDOE")%>%
        # Split on new lines 
        str_split(pattern = "\n", simplify = TRUE)
        
    # Some pdfs may be more than one page, so loop over all pages
    # Atlantic County will only run this loop once
    for(i in 1:dim(tbl)[1]){
    
        # Find the county name and save name to object 
        county_cell <- stringr::str_which(tbl[i,], "County")
        county_name <- tbl[i,county_cell]%>%
        stringr::str_squish()

        # Find the senate candidates cell 
        # Pull out candidates, turn them into a vector 
        # Keep only the last names 
        candidates_cell <- stringr::str_which(tbl[i,], "Robert")
        candidates <- tbl[i, candidates_cell] %>% 
            trimws()%>%
            str_squish()%>%
            str_replace_all(pattern = " R. ", " ")%>%
            str_replace_all(pattern = " Lynn ", " ")%>%
            str_split(pattern = "\\s")
        candidates <- unlist(candidates)
        
        # Because each candidate only has two names, after we remove the 
        # initials R recycles and keeps every other cell, which is last names 
        candidates <- candidates[c(FALSE, TRUE)]
        
        # Find the party cell. The actual table starts after this one
        party_cell <- stringr::str_which(tbl[i,], "Democratic")
       
        # Start the data frame 
        table_start <- party_cell + 1
        
        # Find the line with totals. The last line of interest is 
        # directly before it 
        table_end <- stringr::str_which(tbl[i,], "NJDOE")[1] -1
    
        # Subset to the table of interest 
        table <- tbl[i, table_start:table_end]
        
        # Create a delimiter everywhere there are 2 spaces 
        table <- str_replace_all(table, "\\s{2,}", "|")
        
        # Now we can pull out the data
        
        # Make a text connection and read that in as a dataframe
        text_con <- textConnection(table)
        
        df <- read.csv(text_con, sep = "|", header = F, stringsAsFactors = F)%>%
            dplyr::select_if(not_all_na)
        # Put in the appropriate column names and then add US senate as office 
        colnames(df)<- c("precinct", candidates)
        df <- df %>% 
            mutate(office = "US Senate")%>%
            mutate(county = county_name)%>%
            select(county, precinct, office, everything())
        tables[[i]] <- df
    }
    if(length(tables)==1){
        out <- tables[[1]]
    }else{
       out <- dplyr::bind_rows(tables) 
    }
    out
}
```

Supposing that we have stored all the 2018 Senate pdf urls in a vector called senate_urls, we can then make use of purrr::map and purrr::map_dfr() functions to run each through pdf_text and then our function, followed by tidyr::gather() to get our data into a long format. 


```r
# pdf_text() and map to the rescue

urls <- c("https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-atlantic.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-bergen.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-burlington.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-camden.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-capemay.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-cumberland.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-essex.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-gloucester.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-hudson.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-hunterdon.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-mercer.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-middlesex.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-monmouth.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-morris.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-ocean.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-passaic.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-salem.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-somerset.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-sussex.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-union.pdf",
                 "https://www.state.nj.us/state/elections/assets/pdf/election-results/2018/2018-general-election-results-us-senate-warren.pdf")

senate_urls <- map(urls, pdf_text)

nj_senate <- senate_urls %>% 
    map_dfr(senate_clean_table)%>%
    gather(candidate, votes, -county, -precinct, -office)
```

We can now take at our now useful voting data. 


```r
head(nj_senate)
```

```
##            county         precinct    office candidate votes
## 1 Atlantic County     Absecon City US Senate  Menendez  1551
## 2 Atlantic County    Atlantic City US Senate  Menendez  6039
## 3 Atlantic County  Brigantine City US Senate  Menendez  1391
## 4 Atlantic County       Buena Boro US Senate  Menendez   583
## 5 Atlantic County Buena Vista Twp. US Senate  Menendez  1095
## 6 Atlantic County      Corbin City US Senate  Menendez    77
```
