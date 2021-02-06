---
title: An Example of IV Estimation in R
author: alex
date: '2021-02-06'
slug: []
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2021-02-06T12:28:04-08:00'
featured: no
image:
  placement: 1
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

There have likely been more words written about the use and misuse of instrumental variables than exist atoms in the universe. When I was starting out in grad school, almost all of our methods education came in the context of experiments. Instrumental Variables were treated like a compliance problem. A researcher ran an experiment, but some people decided not to comply with treatment for some reason which led to missing values. By using the random assignment as an instrument for treatment, the researcher could find the Complier Average Treatment Effect (CATE). Without doing anything other than analyzing the experiment based on the intention to treat (ITT), the researcher would get an estimate that would be smaller and noisier. The CATE can be defined as: 

$$ \begin{aligned}
CATE = \frac{ITT}{Pr(compliers)}
\end{aligned} $$ 


Importantly, the CATE does not equal what the researcher is really interested in, which is the Average Treatment Effect (ATE) unless the researcher got very lucky and the ATE for non-compliers is identical to the CATE. Incidentally, this probability has measure zero at least practically speaking. 

Of course, most of the time IV is considered in the case on non-experimental data for which the researcher plans to use a regression analysis and where there is a worry about an endogenous regressor. In this context, regression estimates will measure only the magnitude of association, but not the magnitude and direction of causation needed. Not great! 

In this context, IV a common strategy to get the Local Average Treatment Effect (LATE), for which the most common estimation strategy is two stage least squares (2SLS). However, we can still use the reduced form division estimator and it is sometimes useful for pedagogical reasons. Consider the following example from [Cameron and Trevedi (2005).](https://www.google.com/books/edition/Microeconometrics/Zf0gCwxC9ocC?hl=en&gbpv=0)

Suppose there is a data generating process (DGP) defined as follows:

$$ \begin{aligned}
Y &= 0.5X + u
\end{aligned} $$

$$ \begin{aligned}
X &= Z + v 
\end{aligned} $$

where z is $ N(2,1) $ and $ (u,v) $ are jointly standard normal with a correlation between them of $ \rho = 0.8 $. In R, we can rely on a formula for bivariate joint normality to simulate this scenario. 


```r
set.seed(123)
makeBivariateNormalVars <- function(N, mu_x, mu_y, 
                                    sigma_x, sigma_y,
                                    rho){
  # Begin with two independent N(0,1) variables 
  Z1 <- rnorm(N, 0, 1)
  Z2 <- rnorm(N,0,1)
  
  u <- sigma_x*Z1 + mu_x
  v <- sigma_y *(rho*Z1 + sqrt(1-rho^2)*Z2) + mu_y
  return(list(u,v))
}
N <- 10000
errorTerms <- makeBivariateNormalVars(10000, 0, 0, 1, 1, .8)
u <- errorTerms[[1]]
v <- errorTerms[[2]]
z <- rnorm(N, 2,1)
x <- 0 + z + v
y <- 0 + 0.5*x + u
```

The true value of X is 0.5, but the estimate obtained from OLS will be much too large. 


```r
lm(y~x)$coefficient[2] 
```

```
##         x 
## 0.9037703
```

Yikes. Much much too large. Now let's bring back in our Wald Estimator procedure for IV. 

In our first step, we run the regression of y on z, the exogenous regressor. 


```r
# IV procedure 
# Step 1 
num <- round(lm(y~z)$coefficient[2],3)
```

In our second step, we run the regression of x on z. 


```r
# Step 2 
den <- round(lm(x~z)$coefficient[2],3)
```

In our third step, we get the Wald Estimator for the reduced form equation by dividing Step 1 by Step 2. 


```r
# Step 3 
# IV estimator 
num/den 
```

```
##         z 
## 0.5197239
```

A substantially closer estimate, though not exactly .5. This is ok though because an IV estimator is not unbiased but is consistent. That means if we increase N, the estimator will coverge in probability to the parameter of interest. 

To prove it, you could run a simulation with different size N like the one coded below. I wrapped the procedure in this example into a function called `simulate`. 


```r
simulate <- function(N){
  errorTerms <- makeBivariateNormalVars(N, 0, 0, 1, 1, .8)
  u <- errorTerms[[1]]
  v <- errorTerms[[2]]
  z <- rnorm(N, 2,1)
  x <- 0 + z + v
  y <- 0 + 0.5*x + u
  num <- lm(y~z)$coefficient[2]
  den <- lm(x~z)$coefficient[2]
  return(num/den)
}

set.seed(1234)
Ns <- seq(1000, 100000, by = 10000)

test <-purrr::modify(Ns, simulate)

df <- data.frame(N = Ns, value = test)
p <- ggplot2::ggplot(df, ggplot2::aes(Ns, value))+
  ggplot2::geom_line()+
  ggplot2::ylim(0.45, 0.55)+
  ggplot2::geom_hline(yintercept =  .5, color = "blue")+
  ggplot2::theme_bw()+
  ggplot2::xlab("Sample Size")+
  ggplot2::ylab("Parameter Estimate")+
  ggplot2::ggtitle("Wald Estimation with different sample sizes")
```
