---
title: 'Notes on Bulk Recoding in R '
author: R package build
date: '2021-04-22'
slug: []
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: [alex]
lastmod: '2021-04-22T15:05:52-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


In the spirit of notes to myself, here's a neat trick I learned to bulk recode lots of variables at once. 

Suppose we have conducted a survey experiment and gotten lots of data from our participants. Our raw data looks like the following: 


```r
raw_data <-tibble(
  ID = LETTERS[1:15],
  var1 = sample(1:15, replace = F),
  var2 = sample(16:30, replace = F),
  var3 = sample(31:45, replace = F)
)
kable(raw_data)
```



|ID | var1| var2| var3|
|:--|----:|----:|----:|
|A  |    1|   17|   40|
|B  |    5|   18|   38|
|C  |   15|   24|   41|
|D  |    9|   28|   36|
|E  |   10|   26|   45|
|F  |    4|   19|   44|
|G  |    2|   20|   34|
|H  |   12|   22|   39|
|I  |   13|   27|   42|
|J  |   11|   23|   32|
|K  |    7|   25|   35|
|L  |   14|   30|   43|
|M  |    8|   29|   31|
|N  |    3|   21|   37|
|O  |    6|   16|   33|

Because survey data is likely to contain errors, we have a subject matter expert look at the data. They come back with several changes that we need to make to the data. Helpfully, they put them into a data frame like the following. 


```r
fixes <- tibble(
  ID = c("A", "D", "F", "G"),
  original_var = c("var1", "var2", "var2", "var3"),
  original_response = as.integer(c(1, 28, 19, 34)),
  correct_response = as.integer(c(1000, 2800, 1900, 3400))
)
```


```r
kable(fixes)
```



|ID |original_var | original_response| correct_response|
|:--|:------------|-----------------:|----------------:|
|A  |var1         |                 1|             1000|
|D  |var2         |                28|             2800|
|F  |var2         |                19|             1900|
|G  |var3         |                34|             3400|

Here we have a problem of updating several variables based on a set of (potentially different) conditions. Here, each variable needs to be updated based on an ID, but we do not need to update every variable/ID combination. 

The brute force way of solving this problem (in the tidyverse) might look something like the following: 


```r
fix_data_b <- raw_data %>% 
  mutate(var1 = case_when(
              ID == "A"~as.integer(1000),
              TRUE~var1),
          var2 = case_when(
              ID == "D"~as.integer(2800),
              ID == "F"~as.integer(1900),
              TRUE~var2
        ),
          var3 = case_when(
              ID == "G"~as.integer(3400),
              TRUE~var3
        )
    )
```


```r
kable(fix_data_b)
```



|ID | var1| var2| var3|
|:--|----:|----:|----:|
|A  | 1000|   17|   40|
|B  |    5|   18|   38|
|C  |   15|   24|   41|
|D  |    9| 2800|   36|
|E  |   10|   26|   45|
|F  |    4| 1900|   44|
|G  |    2|   20| 3400|
|H  |   12|   22|   39|
|I  |   13|   27|   42|
|J  |   11|   23|   32|
|K  |    7|   25|   35|
|L  |   14|   30|   43|
|M  |    8|   29|   31|
|N  |    3|   21|   37|
|O  |    6|   16|   33|

A brute force solution is not so bad with only four fixes. What if we added another 25 fixes? Very quickly, this solution breaks down. Besides, there is a lot of unneeded typing, which increases the likelihood of making a mistake. Finally, there's no way to show off that we have a neat trick to make people think we are brilliant, when in fact, we are very lazy. 

The trick I learned to deal with this problem is to realize that this problem is the equivalent of updating a table based on a lookup table. As long as our data frames are in the right shape, we can make these updates all at once instead of having to type out each individually. 

### Step 1: Convert our raw data from wide to long. 


```r
raw_data_l <- raw_data %>% 
  pivot_longer(cols = -ID,
               names_to = "var_name",
               values_to = "original_value")
kable(head(raw_data_l))
```



|ID |var_name | original_value|
|:--|:--------|--------------:|
|A  |var1     |              1|
|A  |var2     |             17|
|A  |var3     |             40|
|B  |var1     |              5|
|B  |var2     |             18|
|B  |var3     |             38|

### Step 2: Join our lookup table to our converted raw data. 


```r
raw_data_l %>% 
  left_join(fixes, by = c("ID", "var_name" = "original_var"))%>%
  head()%>%
  kable()
```



|ID |var_name | original_value| original_response| correct_response|
|:--|:--------|--------------:|-----------------:|----------------:|
|A  |var1     |              1|                 1|             1000|
|A  |var2     |             17|                NA|               NA|
|A  |var3     |             40|                NA|               NA|
|B  |var1     |              5|                NA|               NA|
|B  |var2     |             18|                NA|               NA|
|B  |var3     |             38|                NA|               NA|

Note that we now a lot of NAs for each variable that does not require a correction. This is good! We can take advantage of that in Step 3. 

### Step 3: Make the computer do the work of recoding the variables. 


```r
raw_data_l %>% 
  left_join(fixes, by = c("ID", "var_name" = "original_var"))%>%
  mutate(corrected_value = if_else(is.na(original_response),
                                   original_value,
                                   correct_response))%>%
  select(ID, var_name, original_value, corrected_value)%>%
  filter(ID %in% fixes$ID)%>%
  kable()
```



|ID |var_name | original_value| corrected_value|
|:--|:--------|--------------:|---------------:|
|A  |var1     |              1|            1000|
|A  |var2     |             17|              17|
|A  |var3     |             40|              40|
|D  |var1     |              9|               9|
|D  |var2     |             28|            2800|
|D  |var3     |             36|              36|
|F  |var1     |              4|               4|
|F  |var2     |             19|            1900|
|F  |var3     |             44|              44|
|G  |var1     |              2|               2|
|G  |var2     |             20|              20|
|G  |var3     |             34|            3400|

We can put all three steps together in a pipe workflow.


```r
raw_data_l %>% 
  left_join(fixes, by = c("ID", "var_name" = "original_var"))%>%
  mutate(corrected_value = if_else(is.na(original_response),
                                   original_value,
                                   correct_response))%>%
  select(-original_value, -correct_response, -original_response)%>%
  pivot_wider(names_from = "var_name",
              values_from = "corrected_value")%>%
  kable()
```



|ID | var1| var2| var3|
|:--|----:|----:|----:|
|A  | 1000|   17|   40|
|B  |    5|   18|   38|
|C  |   15|   24|   41|
|D  |    9| 2800|   36|
|E  |   10|   26|   45|
|F  |    4| 1900|   44|
|G  |    2|   20| 3400|
|H  |   12|   22|   39|
|I  |   13|   27|   42|
|J  |   11|   23|   32|
|K  |    7|   25|   35|
|L  |   14|   30|   43|
|M  |    8|   29|   31|
|N  |    3|   21|   37|
|O  |    6|   16|   33|

Now we have a solution that will scale to many recodes without typing out much code. All that is required is that we create our lookup table. Since we have to keep track of our recoded changes, either in our code or in a separate document, we might as well leverage that work to make our life easier.   
