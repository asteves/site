---
title: 'OLS Estimation by "Hand" in R '
author: alex
date: '2019-06-20'
slug: ols-estimation-by-hand-in-r
categories: []
tags: [R]
subtitle: ''
summary: 'A common programming assignment when learning regression is to calculate OLS estimators by hand. In this post, I show exactly how to program OLS estimation in R. In addition, I explain how to add different standard error calculations to replicate Huber-White standard errors and Stata robust standard errors. '
authors: []
lastmod: '2019-06-20T14:23:40-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Introduction 
--------------

A common programming assignment when learning regression is to calculate OLS estimators by hand. In this post, I show exactly how to program OLS estimation in R. In addition, I explain how to add different standard error calculations to replicate Huber-White standard errors and Stata robust standard errors. 

When I learn new concepts, I often like to see the completed project first to know what I am working towards. In that spirit, let's simulate some data and show how our OLS calculation works. 





```r
set.seed(42)

# Making a variable for number of observations 
# is helpful if we want to change our dataset
# in the future
N <- 100

x <- round(runif(N, 0,1),2)

# Simulate potential outcomes 
# We will assume constant treatment effect
y0 <- round(rnorm(N, mean = x),2)
y1 <- y0 + 0.35 
z <- sample(0:1, 100, replace = T, prob = c(0.5, 0.5))
y <- ifelse(z, y1, y0)

# To demonstrate missing data handling 
# generate missing data 
miss <- sample(0:1,100, replace = T, prob = c(0.1, 0.9))
x_miss <- ifelse(miss, NA, x)

# Bring it all together in one data frame
dat <- data.frame(x,y0, y1, z, y, x_miss)
head(dat)
```

```
##      x   y0   y1 z    y x_miss
## 1 0.91 1.23 1.58 1 1.58     NA
## 2 0.94 0.16 0.51 1 0.51     NA
## 3 0.29 1.87 2.22 1 2.22     NA
## 4 0.83 1.47 1.82 1 1.82     NA
## 5 0.64 0.73 1.08 0 0.73     NA
## 6 0.52 0.80 1.15 0 0.80     NA
```

Now that we have our data, we pass our regressions specification to our (soon to be written) function. We can also check to confirm that our (soon to be written) function effectively handles missing data. 


```r
ols(y = "y", x = c("z", "x"), data = dat, se_type = "classical", ci=TRUE)
```

```
##             Estimate        SE    t-value   p-val   conf.low conf.high df
## intercept -0.1471975 0.2060169 -0.7144921 0.47664 -0.5560841 0.2616891 97
## z          0.1300179 0.1856421  0.7003683 0.48537 -0.2384304 0.4984661 97
## x          1.4589214 0.3080066  4.7366562 0.00001  0.8476135 2.0702292 97
```

```r
ols(y = "y", x = c("z", "x_miss"), data = dat, se_type = "classical", ci = TRUE)
```

```
##             Estimate        SE    t-value   p-val   conf.low conf.high df
## intercept -1.1696384 0.5300050 -2.2068440 0.07842 -2.5320597 0.1927829  5
## z         -0.5197353 0.6318404 -0.8225738 0.44819 -2.1439327 1.1044620  5
## x_miss     3.6392306 1.0549444  3.4496895 0.01824  0.9274097 6.3510515  5
```

OLS Estimators
--------------

Our goal is to estimate a population model that includes one or more independent variables. 

\\[y = \beta_0 + \beta_1x_1 + \beta_2x\_2 + ... + \beta_kx\_k + u\\]

As shorthand we refer to parameters other than the intercept as slope parameters. Our goal is to get estimates \\[\beta_0, \beta_1,...,\beta_k\\] We choose these to minimize the sum of squared residuals. This minimization leads to k + 1 linear equations in k + 1 unknowns. We call these the OLS first order equations. More detailed notes on OLS regression are available [here](https://www.alexstephenson.me/courses/econometrics/chap3/).

Using our simulated data set called dat, we want to estimate the regression y on x and z. Since we are calculating by hand, the easiest representation comes by using linear algebra. Therefore, the first step is to get our data into matrix form. We do that in R as follows: 


```r
y <- as.matrix(dat[, "y"])
x <- as.matrix(dat[, c("x","z")])
```

An important computational point for OLS is to add an intercept column that is N x 1 matrix of 1's. 


```r
intercept <- rep(1, nrow(x))

# Coerce to matrices 
Y <- as.matrix(y)
X <- as.matrix(cbind(intercept, x))
```

In matrix notation, the OLS estimator is \\[(X'X)^{-1}X'y\\]

In R: 


```r
# Store the first part as a matrix called A 
# This isn't strictly necessary but it's convenient 
A <- solve(t(X) %*% X)

# Find our OLS Beta coefficient estimates 
betas <- A %*% t(X) %*% Y
betas
```

```
##                 [,1]
## intercept -0.1471975
## x          1.4589214
## z          0.1300179
```

Standard Errors 
--------------

Thus far, we have estimated point estimates for our model parameters. For obvious reasons, we should believe that our estimates have variance. Therefore the next part of our computation is to calculate standard errors. To do this, we first calculate the covariance matrix of our regression estimate and then take the square root of the diagonal to get the standard errors for each estimator. 


```r
# Calculate residual variance
residuals <- Y - X %*% betas

# Number of variables for degree of freedom calculation 
p <- ncol(X) - 1 

df <- nrow(X) - p - 1 

# Residual variance 
res_var <- sum(residuals^2) / df 

# Calculate covariance matrix of estimate 
beta_cov <- res_var * A 

# Square root of the diag 
beta_se <- sqrt(diag(beta_cov))

beta_se 
```

```
## intercept         x         z 
## 0.2060169 0.3080066 0.1856421
```

Turning our Work into a Function
--------------

Thus far, we have been writing a script step by step using one data set. Suppose we want to run a regression model on a different dataset. This will be a pain and require us to change a lot of code, which increases the probability that we will make an error and introduce a bug. A much better strategy is to build a function that includes our code. That way, we follow the principle of "Do not Repeat Yourself" as well as making it easier to change our code or add extensions in the future. Functions tend to be easier to test. 

Our function will have arguments for data, standard error calculation, and confidence interval. The function outputs a data frame, which I find the most convenient form of output for making results tables.


```r
ols <- function(y,
                x,
                data,
                se_type = "classical",
                ci = F) {
  
  ### Variable definitions 
  # y: <string> data frame column for our dependent variable 
  # x: <string> data frame columns for our independent variable.
  ## Passed to function as a vection 
  # data: <data frame> A data frame of our data 
  # se_type: <string> our standard error type. 
  ## We can choose from c("classical", "HC0",
  ## "stata"). Defaults to "classical"
  # ci: <logical> flag to include confidence interval. Defaults to F 
  
  
    # Step 1: Calculate OLS Estimators our "Beta Hats"

    ## Make sure we have no missing data
    data <- data[complete.cases(data[, c(y,x)]),]
    # Select our dependent and independent variables
    y <- data[, y]
    x <- data[, x]
    intercept <- rep(1, nrow(x))
    Y <- as.matrix(y)
    X <- as.matrix(cbind(intercept, x))
    # We'll this part multiple times so store it as a matrix
    # Call this matrix A since every matrix seems to be
    # called A
    A <- solve(t(X) %*% X)

    # Our Beta Coefficient estimates
    betas <- A %*% t(X) %*% Y

    # Residuals from our estimates
    residuals <- Y - X %*% betas
    # Step 2: Calculate residual variance

    # Number of columns
    p <- ncol(X) - 1

    # Degrees of freedom calculation
    df <- nrow(X) - p - 1
    if (se_type == "classical") {
        # equivalent to (t(residuals) %*% residuals)/df
        res_var <- sum(residuals ^ 2) / df

        # Covariance matrix of estimated regression
        # coefficients
        # e.g eX'X
        beta_cov <- res_var * A

        # take square root of the diagonal of matrix
        # to get sd
        beta_se <- sqrt(diag(beta_cov))
    }
    if (se_type == "HC0") {
        # Replicating HC0 errors
        u2 <- residuals ^ 2
        XDX <- 0
        for (i in 1:nrow(X)) {
            XDX <- XDX + u2[i] * X[i, ] %*% t(X[i, ])
        }
        # A%*% XDX %*% A
        # XDX is the same size matrix as A
        varcov_m <- A %*% XDX %*% A

        beta_se <- sqrt(diag(varcov_m))

    }
    if (se_type == "stata") {
        u2 <- residuals ^ 2
        XDX <- 0
        for (i in 1:nrow(X)) {
            XDX <- XDX + u2[i] * X[i, ] %*% t(X[i, ])
        }
        correction <- nrow(X) / (nrow(X) - p - 1)
        varcov_m <- correction * A %*% XDX %*% A
        beta_se <- sqrt(diag(varcov_m))
    }

    t_val <- betas / beta_se

    # For now just calculating two tailed
    p_v <- round(2 * pt(abs(t_val), df, lower = FALSE), 5)

    if (ci) {
        conf.low <- betas - qt(0.05 / 2, df, lower.tail = FALSE) * beta_se
        conf.high <-
            betas + qt(0.05 / 2, df, lower.tail = FALSE) * beta_se
        out <-
            data.frame(betas, beta_se, t_val, p_v, conf.low, conf.high, df)
        names(out) <-
            c("Estimate",
              "SE",
              "t-value",
              "p-val",
              "conf.low",
              "conf.high",
              "df")
        return(out)
    } else{
        # Step 3: Output as a data frame
        out <- data.frame(betas, beta_se, t_val, p_v)
        names(out) <- c("Estimate", "SE", "t-value", "p-val")
        return(out)
    }

}
```


The completed function extends our standard error functionality to replicate Huber White standard errors and Stata's robust standard error calculation. 


```r
ols(y = "y", x = c("z", "x"), data = dat, se_type = "HC0", ci=TRUE)
```

```
##             Estimate        SE    t-value   p-val   conf.low conf.high df
## intercept -0.1471975 0.1616507 -0.9105898 0.36477 -0.4680294 0.1736344 97
## z          0.1300179 0.1825116  0.7123812 0.47794 -0.2322172 0.4922529 97
## x          1.4589214 0.2816168  5.1805191 0.00000  0.8999899 2.0178528 97
```

```r
ols(y = "y", x = c("z", "x"), data = dat, se_type = "stata", ci=TRUE)
```

```
##             Estimate        SE    t-value   p-val   conf.low conf.high df
## intercept -0.1471975 0.1641314 -0.8968269 0.37203 -0.4729529 0.1785580 97
## z          0.1300179 0.1853125  0.7016141 0.48460 -0.2377761 0.4978119 97
## x          1.4589214 0.2859386  5.1022196 0.00000  0.8914125 2.0264303 97
```

A Note for Actual Work  
--------------

In actual research, there is no reason to ever calculate OLS by hand because R comes with the built in function lm(). For other common linear estimators packages like [estimatr](https://declaredesign.org/r/estimatr/) provide consistent interfaces for a range of commonly-used linear estimators. These packages are written in C++, which also make them substantially faster than our present function. 
