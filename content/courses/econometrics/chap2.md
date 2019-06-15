---
title: "Simple Linear Regression"
linktitle: Simple Linear Regression
toc: true
type: docs 
date: "2019-06-05T00:00:00+01:00"
draft: false 
menu:
  econometrics:
    parent: Econometrics
    weight: 1

weight: 1
---

We are interested in studying models that take the following form 

\\[y = \beta_0 + \beta_1x + u\\]

where \\(\beta_0\\) is the intercept, \\(\beta_1\\) is the slope parameter and u is the error term. In the next set of notes, we will extend this model to situations where we have more than one covariate. 

We can think of \\(\beta_0 + \beta_1x\\) as the systematic part of y whereas u is the unsystematic part of y. That is, u represents y not explained by x. 

## Error Term Assumptions 

In order to make progress, we make the following assumptions about the error term. 

1. \\(E[u] = 0\\) as long as an intercept term is included in the equation. Note that this essentially defines the intercept. 

2. \\(E[u|x] = E[u] = 0\\). This is the Zero Conditional Mean Assumption for the error term. 
 - The average value of the unobservables is the same across all slices of the population determined by the value of x and is equal to the average of u over the entire population 
 - By EA.1 that means the average is 0

## Deriving OLS Estimates 

To estimate the Population Regression Function (PRF), we need a sample. 

Let \\((x_i, y_i): i = 1,...,n\\) be a random sample of size n from the population. We can estimate the PRF by a model: 

\\[y = \beta_0 + \beta_1x_i + u_i\\] (E.2)

Error Assumption 2 implies that in the popuation x and u are uncorrelated, and the zero conditional mean assumption for the error implies that \\(E[u] = 0\\). This implies that the covariance between x and u is 0 or formally: 
\\[Cov(x,u) = E(xu) = 0\\] 

We can rewrite previous equations as follows 

\\[E[u] = E[ y - \beta_0 + \beta_1x]\\] (E.3)


\\[Cov(x,u) = E[x(y - \beta_0 + \beta_1x)]\\] (E.4)

Our goal is to choose sample \\(\hat{\beta_0}\\),\\(\hat{\beta_1}\\) to solve the sample equations: 

\\[\frac{1}{n}\sum_{i=1}^n y - \hat{\beta_0} + \hat{\beta_1}x = 0\\] (E.5)

\\[\frac{1}{n}\sum_{i=1}^n x_i(y - \hat{\beta_0} + \hat{\beta_1}x) = 0\\] (E.6)

Rewrite E.4 

\\[\bar{y} = \hat{\beta_0} + \hat{\beta_1}\bar{x}\\] which implies 

\\[\beta_0 = \bar{y} - \hat{\beta_1}\bar{x}\\]

### Estimating The Slope Parameter

Drop the \\(\frac{1}{n}\\) in E.5 because it does not affect the solution. Plug in \\(\bar{y} - \hat{\beta_1}\bar{x}\\) for \\(\beta_0\\) which yields the equation 

\\[\sum_{i=1}^n x_i(\bar{y} - \hat{\beta_1}\bar{x}) - \hat{\beta_1}x) = 0\\]

Rearrange terms to get the y's and the x's on opposite sides of the equation. 
\\[\sum_{i=1}^n x_i(y_i - \bar{y})\\] 
\\[\hat{\beta_1}\sum x_i(x_i - \bar{x})\\] 

Setting these equal to each other and using properties of the sum operator, we can rewrite the the top sum to be \\(Cov(x,y)\\) and the bottom sum to \\(V(x)\\). As long as \\(V(x) > 0\\), 

\\[\hat{\beta_1} = \frac{\hat{Cov(x,y)}}{\hat{V(x)}}\\]

In words, the slope parameter estimate is the sample covariance of x and y divided by the sample variance of x. We refer to this as the OLS procedure and the OLS regression line as 

\\[\hat{y} = \hat{\beta_0} + \hat{\beta_1}x\\]

## Algebraic Properties of OLS on Any Sample of Data 

The following hold by construction for any sample of data estimated by OLS 

1. The sum and therefore sample average of the residuals is 0. This is because the OLS estimates are chosen to make the residuals sum to 0. 
2. Sample covariance between regressors and OLS residuals is 0 
3. The point \\((\bar{x}, \bar{y})\\) is always on the OLS regression line 

## Variation in Y 

We can view OLS as decomposing each \\(y_i\\) into two parts, a fitted value and a residual. There are three parts of this decomposition: the total sum of squares (SST), the explained sum of squares (SSE), and the residual sum of squares (SSR). 

\\[SST = \sum_{i=1}^n (y_i -\bar{y})^2\\]

\\[SSE = \sum_{i=1}^n (\hat{y_i} -\bar{y})^2\\]

\\[SSR = \sum_{i=1}^n \hat{u}^2\\]

SST is a measure of total sample variation in the \\(y_i\\)'s. Dividing SST by n-1 gets us the sample variance of y. 

The Total Variation in y is SST = SSE + SSR. 

To derive

\\[\sum_{i=1}^n (y_i -\bar{y})^2\\]

\\[\sum_{i=1}^n [(y_i - \hat{y_i}) + (\hat{y_i}-\bar{y})]^2\\]

\\[\sum_{i=1}^n [\hat{u_i} + (\hat{y_i}-\bar{y})]^2\\]

Expand out the sum and replace with definitions to get 

\\[SSR + 2Cov(\hat{u}, \hat{y}) + SSE\\]

Since the covariance between u and y is 0, that term drops out. 

## Goodness of Fit 

The ratio of the explained sample variation in y by x is known as \\(R^2\\) and defined: 

\\[R^2 = 1 - \frac{SSR}{SST}\\]

## Expected Values and Unbiasedness of OLS Estimators

OLS is an unbiased estimator of the population model provided the following assumptions hold. These assumptions are also known as Gauss-Markov assumptions. 

### A1. Linear in paramters

In the population model, y is related to x and u 

\\[y = \beta_0 + \beta_1x + u \\]

### A2. Random Sample

We have a random sample of size n from the population model 

### A3. Sample variation in x 

The sample outcomes \\(x_i: i = 1,2,..., n\\) are not all the same value. If they are, there is no variance of X and so \\(\beta_1\\) cannot be estimated. 

### A4. Zero Conditional Mean of the Error

For a random sample, this assumption implies 

\\[E(u_i|x_i) = 0: \forall i \in [0,1,...n]\\]

A4 is violated whenever we think that u and x are correlated. In the simple bivariate case, an example might be using the variable education to predict salary. education is correlated with many variables, including income and family history. These may affect salary and therefore will give us biased results. 

*Note: We can write the slope estimator \\(\beta_1\\) in a slightly different way* 

\\[\hat{\beta_1} = \frac{\sum\_{i=1}^n (x_i - \bar{x})*(\beta_0 - \beta_1x + u_i)}{SST_x}\\]

\\[\hat{\beta_1} = \beta\_0 \sum\_{i=1}^n\\ + \beta\_1\sum\_{i=1}^n x_i(x_i -\bar{x}) + \sum\_{i=1}^n u_i(x_i - \bar{x})\\] 

The first term sums to 0 and drops out. Thus: 

\\[\hat{\beta_1} = \beta_1 + \frac{\sum\_{i=1}^n u_i(x_i - \bar{x})}{SST_x}\\]

We now have all the information we need to prove that OLS is unbiased. Unbiasedness is a feature of the sampling distributions of \\(\hat{\beta_0}\\) and \\(\hat{\beta_1}\\). Unbiasedness says nothing about the estimates for any *given sample* we may draw.  

### Theorem 1: Using A1-A4 OLS produces unbiased estimates 

\\(E(\hat{\beta_0}) = \beta_0\\) and \\(E(\hat{\beta_1}) = \beta_1\\)for any values of \\(\beta_0\\) and \\(\beta_1\\). 

Proof: 

In this proof the expected values are conditional on sample values of the independent variable x. Because \\(SST_x\\) and \\((x_i - \bar(x))\\) are functions on of \\(x_i\\) they are non-random once we condition on x. 

\\[E[\hat{\beta_1}] = E[\beta_1 + \frac{\sum\_{i=1}^n u_i(x_i - \bar{x})}{SST_x}]\\]

\\[E[\hat{\beta_1}] = \beta_1 + \frac{\sum\_{i=1}^n E[u_i(x_i - \bar{x})]}{SST_x}\\] 

\\[E[\hat{\beta_1}] = \beta_1 + \frac{\sum\_{i=1}^n 0 (x_i - \bar{x})}{SST_x}\\] 

\\[E[\hat{\beta_1}] = \beta_1\\] 

We can also prove the same for \\(\beta_0\\). 

\\[E[\hat{\beta_0}] = \beta_0 + E[(\beta_1 - \hat{\beta_1}\bar{x} + E[\bar{u}]\\] 

\\[E[\hat{\beta_0}] = \beta_0 + E[(\beta_1 - \hat{\beta_1}\bar{x} + 0\\] 

\\[E[\hat{\beta_0}] = \beta_0\\] 

In the last equation, because \\(\hat{\beta_1} = \beta_1\\) the second term drops out. 

## Variances of OLS Estimators 

An additional assumption we can make about the variance of the OLS estimators is that the error u has the same variances conditional on any value of the explanatory variable. 

\\[V(u|x) = \sigma^2\\]

By adding this assumption, which to be clear will break down horribly if it is violated, we can prove the following theorem. 

### Theorem 2: Using assumptions 1-4 and homoskedastic error assumption

\\[V(\hat{\beta_1}) = \frac{\sigma^2}{SST_x}\\] and \\[V(\hat{\beta_0}) = \frac{\sigma^2\frac{1}{n}\sum\_{i=1}^n x_i^2}{\sum\_{i=1}^n(x_i - \bar{x})^2}\\]

where these are conditioned on the sample values. 

Proof for \\(V(\hat{\beta_1})\\)

\\[V(\hat{\beta_1}) = \frac{1^2}{SST_x^2}V(\sum\_{i=1}^n u_i(x_i - \bar{x}))\\]

Substitute \\(d_i = (x_i - \bar{x})\\)

\\[V(\hat{\beta_1}) = \frac{1^2}{SST_x^2}\sum\_{i=1}^n u_i d_i^2\\]

Since \\(V(u_i) = \sigma^2 : \forall i\\) we can substitute that constant into the equation. 

\\[V(\hat{\beta_1}) = \frac{1}{SST_x^2}\sigma^2 \sum\_{i=1}^n d_i^2\\]

Observe that the second RHS term is just \\(SST_x\\) after pulling out the constant, we can rewrite as 

\\[V(\hat{\beta_1}) = \frac{\sigma^2 SST_x}{SST_x^2}\\]

which reduces to our stated result. 

Now that we know the way to estimate the variance, we can ask the following question. How does \\(V(\hat{\beta_1})\\) depend on error variance? 

1. The larger the error variance, the larger \\(V(\hat{\beta_1})\\). 
2. The larger the \\(V(x)\\), the smaller \\(V(\hat{\beta_1})\\)
3. As sample size increases, the total variation in x increases which leads to a decrease in \\(V(\hat{\beta_1})\\)

## Estimating the Error Variance 

Errors are never observed. Instead, we observe residuals that we can compute from our sample data. We can write the errors as a function of the residuals. 

\\[\hat{u_i} = u_i - (\hat{\beta_0} - \beta_0) - (\hat{\beta_1} - \beta_1)x\\]

One problem that we run into is that using the residuals as an estimator is biased without correction because it does not take into account two restrictions for OLS residuals. OLS residuals have to sum to 0 and have a 0 covariance between x and u. Formally, 

\\[\sum\_{i=1}^n \hat{u_i} = 0\\] 

and 

\\[\sum\_{i=1}^n \hat{u_i}x_i = 0\\]. 

Thus we need to correct by n-2 degrees of freedom for an unbiased estimator. When we do so, we get the following. 

\\[\hat{\sigma}^2 = \hat{s}^2 = \frac{1}{n-2}\sum\_{i=1}^n\hat{u_i}^2\\]

\\[\hat{\sigma}^2 = \frac{SSR}{n-2}\\]



