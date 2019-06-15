---
title: "Multiple Linear Regression"
linktitle: Multiple Linear Regression
toc: true
type: docs 
date: "2019-06-14T00:00:00+01:00"
draft: false 
menu:
  econometrics:
    parent: Econometrics
    weight: 2

weight: 2
---

We are now going to extend our previous discussion of Simple Linear Regression into the multiple variate case. Here we consider models that include more than one independent variable. 

\\[y = \beta_0 + \beta_1x_1 + \beta_2x\_2 + ... + \beta_kx\_k + u\\]

where \\(\beta_0\\) is the intercept, \\(\beta_1\\) measures change in y with respect to \\(x_1\\) holding all other covariates fixed and so on. 

As shorthand we refer to parameters other than the intercept as slope parameters. 

We can generalize the zero conditional mean assumption for the errors to be 

\\[E(U|x\_1, x\_2, ..., x\_k) = 0\\]

## Mechanics and Interpretation of OLS

Our goal is to get estimates \\(\beta_0, \beta_1,...,\beta_k\\). We choose these to minimize the sum of squared residuals 

\\[\sum_{i=1}^n\hat{y_i} = \hat{\beta_0} + \hat{\beta_1}x\_{i1}+...+ \hat{\beta_k}x\_{ik}\\]

This minimization leads to k + 1 linear equations in k + 1 unknowns. We call these the OLS first order equations. 

## OLS Fitted Values and Residuals 

For observation i the fitted values are: 

\\[\sum_{i=1}^n\hat{y_i} = \hat{\beta_0} + \hat{\beta_1}x\_{i1}+...+ \hat{\beta_k}x\_{ik}\\]

Recall that OLS minimizes the average square prediction error which says nothing about any given prediction. The residual is generalized from the simple linear regression case to be: 

\\[\hat{u_i} = y_i - \hat{y_i}\\]

### Properties of fitted values and residuals 

1. The sample average of the residuals is 0. \\(\bar{y} = \hat{\bar{y}}\\)
2. Sample covariance between each independent variable and the residuals is 0 by construction 
3. The average point is always on the regression line by construction. 

## Goodness of Fit 

Like before SST = SSE + SSR and \\(R^2 = 1 - \frac{SSR}{SST}\\)

## Expected Value of OLS Estimators 

We restate the assumptions needed for OLS regressions 

1. Linear in Parameters 

The model can be written as \\(y = \beta_0 + \beta_1x\_1 + \.\.\. + \beta_kx\_k\\)

2. Random sampling 

We have a random sample of n observations \\({(x\_{i1}, x\_{i2}, ..., x\_{ik}): i = 1,2,..., n}\\)

3. No perfect collinearity 

In the sample (and therefore population) none of the independent variables is constant and there are no exact linear relationships among the independent variables 

4. Zero Conditional Mean of Error 

\\[E(u|x_1, x_2, ..., x_k) = 0\\]

This implies that none of the independent variables are correlated with the error term 

If these four assumptions hold 

*Theorem 3.1: OLS is unbiased*

\\[E(\hat{\beta_j} = \beta_j : j = 1,2,\.\.\.,k)\\] 

for any values of the population parameter \\(\beta_j\\)

Note: Adding irrelevant independent variables does not effect unbiasedness. These terms will on average be 0 across many samples. Adding irrelevant independent variables **will** hurt the estimator's variances. 

## Omitted Variable Bias 

A major problem that will lead to bias in our estimates is omitting a relevant variable from our model. To see why suppose we have the following population model 

\\[y = \beta_0 + \beta_1x_1 + \beta_2x_2 + u\\]

but we only run the regression 

\\[y = \hat{\beta_0} + \beta_1x\\]

For example, we want to estimate the effect of education on wages but do not include some measure of innate ability. 

Since the model is misspecified, we can define bias as:

\\[\tilde{\beta_1} = \hat{\beta_1} + \hat{\beta_2}\tilde{\delta_1}\\]

where \\(\hat{\beta_1}, \hat{\beta_2}\\) are slope estimator from the regression \\(y_i \\) on \\(x_1, x_2\\) and \\(\tilde{\delta_1}\\) is the slope from the regression \\(x\_{i2}\\) on \\(x\_{i1}\\). 

\\(\tilde{\delta_1}\\) depends only on the independent variables in the sample so we can treat it as a fixed quanitty when computing \\(E[\tilde{\beta_1}]\\)

\\[E[\tilde{\beta_1}] = E[\hat{\beta_1} + \hat{\beta_2}\tilde{\delta_1}]\\]

\\[E[\tilde{\beta_1}] = E[\hat{\beta_1} + \tilde\delta_1\hat{\beta_2}]\\]

\\[E[\tilde{\beta_1}] = \beta_1 + \tilde\delta_1\beta_2\\]

which implies that the omitted variable bias is \\(\beta_2\tilde{\delta_1}\\)

The bias in the model is 0 if: 

1. \\(\beta_2 = 0\\)
2. \\(\tilde{\delta_1}=0\\)

which occurs if and only if \\(x_1\\) and \\(x_2\\) are uncorrelated in the sample. 

## Variance of OLS Estimators 

Keeping our previous Multiple Linear Regression assumptions we add 

Assumption 5: Homoskedasticity of errors 
The error u has the same variance given any values of the explanatory variables. 

\\[V(u| x_1, x_2, ..., x_k)=\sigma^2\\]

*Theorem 2* 
Under assumptions 1-5 conditional on the sample values of the independent variables 

\\[v(\hat{\beta_j}) = \frac{\sigma^2}{{SST}_j(1-R_j^2)} \\]

and 

\\[E[\hat{\sigma%^2}= \sigma^2]\\]

The unbiased estimator of \\(\sigma^2\\) is \\(\hat{\sigma^2}= \frac{1}{n-k-1}\sum_{i=1}^n \hat{u_i}^2 \\) where n is the number of observations and k + 1 is the number of parameters. 

The standard error of \\(\hat{\beta_j}\\) is

\\[se(\hat{\beta_j})= \frac{\hat{\sigma}}{\sqrt{{SST}_j(1-\hat{R_j}^2)}}\\]

## Efficiency Properties of OLS 

The OLS estimator \\(\hat{\beta_j}\\) for \\(\beta_j\\) is BLUE: The Best Linear Unbiased Estimator. 

 - estimator: rule that can be applied to data to generate an estimate 
 - unbiased \\(E[\hat{\beta_j}]=\beta_j \\) for any estimator
 - linear: An estimator is linear if it can be expressed as a linear function of the data on the dependent variable 
 - best: Gauss-Markov holds that for any estimator \\(\tilde{\beta_j}\\) that is linear and unbiased \\(V(\hat{\beta_j}) \leq V(\tilde{\beta_j}) \\). That is the OLS estimator has at least as small if not smaller variance than any other linear unbiased estimator.