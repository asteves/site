---
title: Regression Tables for Lazy People
author: alex
date: '2020-08-15'
slug: regression-tables-for-lazy-people
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2020-08-15T23:38:17-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In the spirit of writing as note-taking, I wanted to share a neat little trick in R for running regressions. By far the most common table in a political science paper is a regression table. Often, researchers run multiple regression specifications and then present them in a singular table. Each regression may have different sets of variables, or one specification will include an interaction effect. This can mean a lot of typing, which can implicitly violate DRY principles for coding. 

I came across a nice use of the `accumulate()` function from the `purrr` package that both speeds up this task, and makes it programmatic for easy replication. Lazy win!



The basic concept behind `accumulate()` is to apply a function recursively over a list starting from the left. If you want to do so in reverse, then use `accumulate_right()`. Given the name, the main use of this function is for cumulative sums, but we can take advantage of the character formula ability of R to run different specifications. 

In order to replicate this post on your own machine, you will need the following packages. 


```r
install.packages(c("AER", "tidyverse", "estimatr", "texreg"))
```

## A fake data example 

In the first demonstration, I create a simple dataset with three predictors, two potential outcomes, and the observed outcome based on a treatment variable. 

By construction, `\(\tau = 1\)` and the data generating process includes multiple pre-treatment covariates and an interaction. The dataset also includes two useless variables. In experimental or administrative datasets, there are often a large number of covariates that are unused in regression specifications. 


```r
set.seed(42)
N = 1000
dat <- tibble(
  x1 = rnorm(N),
  x2 = rnorm(N, 1, 10),
  x3 = rnorm(N, 10, 3),
  z = sample(c(rep(1,N/2), rep(0,N/2)), N, replace = F),
  y0 = x1 + x2 + x3 + x2*x3+runif(N),
  y1 = y0 + 1,
  yobs = ifelse(z, y1, y0),
  useless1 = runif(N),
  useless2 = runif(N)
) 
```

Now, we can pass our columns of interest and let accumulate do the work. 

```r
cols_to_use <- c("z", "x1", "x2", "x3", 'x2*x3')
predictors <- accumulate(cols_to_use, function(a,b){paste(a,b, sep=" + ")})
formulas <- paste("yobs~", predictors)

print(formulas)
```

```
## [1] "yobs~ z"                        "yobs~ z + x1"                  
## [3] "yobs~ z + x1 + x2"              "yobs~ z + x1 + x2 + x3"        
## [5] "yobs~ z + x1 + x2 + x3 + x2*x3"
```

Neat! We now get our formula specifications in the right order for our table. Using `lm_robust()`, we can now estimate our regression. Unsurprisingly, the model with the interaction perfectly estimates `\(\tau\)`. 


```r
# Functional programming with purrr using map()
# lm_robust requires that we coerce our character to a formula object 
formulas %>% 
  map(~lm_robust(as.formula(.x), data = dat, se_type = "stata"))%>%
  htmlreg(include.ci = F)
```

<table class="texreg" style="margin: 10px auto;border-collapse: collapse;border-spacing: 0px;caption-side: bottom;color: #000000;border-top: 2px solid #000000;">
<caption>Statistical models</caption>
<thead>
<tr>
<th style="padding-left: 5px;padding-right: 5px;">&nbsp;</th>
<th style="padding-left: 5px;padding-right: 5px;">Model 1</th>
<th style="padding-left: 5px;padding-right: 5px;">Model 2</th>
<th style="padding-left: 5px;padding-right: 5px;">Model 3</th>
<th style="padding-left: 5px;padding-right: 5px;">Model 4</th>
<th style="padding-left: 5px;padding-right: 5px;">Model 5</th>
</tr>
</thead>
<tbody>
<tr style="border-top: 1px solid #000000;">
<td style="padding-left: 5px;padding-right: 5px;">(Intercept)</td>
<td style="padding-left: 5px;padding-right: 5px;">17.34<sup>***</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">17.39<sup>***</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">9.54<sup>***</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">-5.90</td>
<td style="padding-left: 5px;padding-right: 5px;">0.46<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(4.81)</td>
<td style="padding-left: 5px;padding-right: 5px;">(4.82)</td>
<td style="padding-left: 5px;padding-right: 5px;">(1.21)</td>
<td style="padding-left: 5px;padding-right: 5px;">(6.12)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.03)</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">z</td>
<td style="padding-left: 5px;padding-right: 5px;">6.94</td>
<td style="padding-left: 5px;padding-right: 5px;">6.91</td>
<td style="padding-left: 5px;padding-right: 5px;">1.62</td>
<td style="padding-left: 5px;padding-right: 5px;">0.91</td>
<td style="padding-left: 5px;padding-right: 5px;">0.97<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(7.18)</td>
<td style="padding-left: 5px;padding-right: 5px;">(7.19)</td>
<td style="padding-left: 5px;padding-right: 5px;">(1.99)</td>
<td style="padding-left: 5px;padding-right: 5px;">(1.99)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.02)</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">x1</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">1.63</td>
<td style="padding-left: 5px;padding-right: 5px;">0.57</td>
<td style="padding-left: 5px;padding-right: 5px;">0.67</td>
<td style="padding-left: 5px;padding-right: 5px;">0.99<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(3.66)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.97)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.97)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.01)</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">x2</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">11.06<sup>***</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">11.07<sup>***</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">1.00<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.17)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.17)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.00)</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">x3</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">1.58<sup>**</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">1.00<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.60)</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.00)</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">x2:x3</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">1.00<sup>***</sup></td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
<td style="padding-left: 5px;padding-right: 5px;">(0.00)</td>
</tr>
<tr style="border-top: 1px solid #000000;">
<td style="padding-left: 5px;padding-right: 5px;">R<sup>2</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">0.00</td>
<td style="padding-left: 5px;padding-right: 5px;">0.00</td>
<td style="padding-left: 5px;padding-right: 5px;">0.92</td>
<td style="padding-left: 5px;padding-right: 5px;">0.93</td>
<td style="padding-left: 5px;padding-right: 5px;">1.00</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">Adj. R<sup>2</sup></td>
<td style="padding-left: 5px;padding-right: 5px;">-0.00</td>
<td style="padding-left: 5px;padding-right: 5px;">-0.00</td>
<td style="padding-left: 5px;padding-right: 5px;">0.92</td>
<td style="padding-left: 5px;padding-right: 5px;">0.93</td>
<td style="padding-left: 5px;padding-right: 5px;">1.00</td>
</tr>
<tr>
<td style="padding-left: 5px;padding-right: 5px;">Num. obs.</td>
<td style="padding-left: 5px;padding-right: 5px;">1000</td>
<td style="padding-left: 5px;padding-right: 5px;">1000</td>
<td style="padding-left: 5px;padding-right: 5px;">1000</td>
<td style="padding-left: 5px;padding-right: 5px;">1000</td>
<td style="padding-left: 5px;padding-right: 5px;">1000</td>
</tr>
<tr style="border-bottom: 2px solid #000000;">
<td style="padding-left: 5px;padding-right: 5px;">RMSE</td>
<td style="padding-left: 5px;padding-right: 5px;">113.49</td>
<td style="padding-left: 5px;padding-right: 5px;">113.53</td>
<td style="padding-left: 5px;padding-right: 5px;">31.38</td>
<td style="padding-left: 5px;padding-right: 5px;">31.02</td>
<td style="padding-left: 5px;padding-right: 5px;">0.29</td>
</tr>
</tbody>
<tfoot>
<tr>
<td style="font-size: 0.8em;" colspan="6"><sup>***</sup>p &lt; 0.001; <sup>**</sup>p &lt; 0.01; <sup>*</sup>p &lt; 0.05</td>
</tr>
</tfoot>
</table>

## An Application with Real Data 

While none of the mechanics change from simulated data to real data (the point!), I find it sometimes helpful to show a technique against actual data. To do so, let's use data on California schools available in the `AER` package. For this example, we are going to do some basic cleaning of the dataset to get a variable for the student-teacher ratio (STR), and average test score (score). In addition, we will create two binary variables for a high student-teacher ratio (HiSTR) and a high percentage of English learners in the schools (HiEL). 



```r
data("CASchools")

# Make use of dplyr's helpful data munging operations 
CASchools <- CASchools %>% 
  mutate(STR = students/teachers,
         score = (read + math)/2,
         HiSTR = as.numeric(STR >=20),
         HiEL = as.numeric(english >= 10))

# Exactly as before, but using the variables in the dataset
cols_to_use <- c("HiSTR", "HiEL", "HiSTR*HiEL")
predictors <- accumulate(cols_to_use, function(a,b){
  paste(a,b,sep = "+")
})

formulas  <- paste("score~", predictors)
```


```r
formulas %>%
  map(~lm_robust(as.formula(.x), data = CASchools, se_type = "stata"))%>%
  htmlreg(include.ci = F)
```

```
## <table class="texreg" style="margin: 10px auto;border-collapse: collapse;border-spacing: 0px;caption-side: bottom;color: #000000;border-top: 2px solid #000000;">
## <caption>Statistical models</caption>
## <thead>
## <tr>
## <th style="padding-left: 5px;padding-right: 5px;">&nbsp;</th>
## <th style="padding-left: 5px;padding-right: 5px;">Model 1</th>
## <th style="padding-left: 5px;padding-right: 5px;">Model 2</th>
## <th style="padding-left: 5px;padding-right: 5px;">Model 3</th>
## </tr>
## </thead>
## <tbody>
## <tr style="border-top: 1px solid #000000;">
## <td style="padding-left: 5px;padding-right: 5px;">(Intercept)</td>
## <td style="padding-left: 5px;padding-right: 5px;">657.25<sup>***</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">664.69<sup>***</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">664.14<sup>***</sup></td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.25)</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.25)</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.39)</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">HiSTR</td>
## <td style="padding-left: 5px;padding-right: 5px;">-7.17<sup>***</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">-3.48<sup>*</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">-1.91</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.83)</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.55)</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.93)</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">HiEL</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">-19.76<sup>***</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">-18.32<sup>***</sup></td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">(1.59)</td>
## <td style="padding-left: 5px;padding-right: 5px;">(2.33)</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">HiSTR:HiEL</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">-3.26</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">&nbsp;</td>
## <td style="padding-left: 5px;padding-right: 5px;">(3.12)</td>
## </tr>
## <tr style="border-top: 1px solid #000000;">
## <td style="padding-left: 5px;padding-right: 5px;">R<sup>2</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">0.03</td>
## <td style="padding-left: 5px;padding-right: 5px;">0.29</td>
## <td style="padding-left: 5px;padding-right: 5px;">0.29</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">Adj. R<sup>2</sup></td>
## <td style="padding-left: 5px;padding-right: 5px;">0.03</td>
## <td style="padding-left: 5px;padding-right: 5px;">0.29</td>
## <td style="padding-left: 5px;padding-right: 5px;">0.29</td>
## </tr>
## <tr>
## <td style="padding-left: 5px;padding-right: 5px;">Num. obs.</td>
## <td style="padding-left: 5px;padding-right: 5px;">420</td>
## <td style="padding-left: 5px;padding-right: 5px;">420</td>
## <td style="padding-left: 5px;padding-right: 5px;">420</td>
## </tr>
## <tr style="border-bottom: 2px solid #000000;">
## <td style="padding-left: 5px;padding-right: 5px;">RMSE</td>
## <td style="padding-left: 5px;padding-right: 5px;">18.74</td>
## <td style="padding-left: 5px;padding-right: 5px;">16.06</td>
## <td style="padding-left: 5px;padding-right: 5px;">16.06</td>
## </tr>
## </tbody>
## <tfoot>
## <tr>
## <td style="font-size: 0.8em;" colspan="4"><sup>***</sup>p &lt; 0.001; <sup>**</sup>p &lt; 0.01; <sup>*</sup>p &lt; 0.05</td>
## </tr>
## </tfoot>
## </table>
```

As we can see, functional programming principles can make our code easier to read and require less typing. The less we have to type, the less we are likely to accidentally introduce a mistake into our work. In addition, the more we get to be lazy, and being lazy is wonderful. 
