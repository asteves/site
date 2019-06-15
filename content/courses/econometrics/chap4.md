---
title: "Linear Regression Inference"
linktitle: Linear Regression Inference
toc: true
type: docs 
date: "2019-06-14T00:00:00+01:00"
draft: false 
menu:
  econometrics:
    parent: Econometrics
    weight: 3

weight: 3
---

From our previous assumptions we add an additional assumption. 

*Assumption 6: Normality*

The population error u is independent of the explanatory variables \\(x_1, x_2, ..., x_k\\) and is normally distributed as \\(u \sim N(0, \sigma^2)\\)

Assumption 6 is strong and amounts to also assuming MLR Assumption 4 and MLR Assumption 5. Taken together, the six assumptions we've made so far collectively are the classical linear model (CLM) assumptions. 

We can summarise the CLM as: 

\\[y|\textbf{x} \sim N(\beta_0 + \beta_1x\_{ik} + ... + \beta_kx\_{ik}, \sigma^2)\\]

where \\(\textbf{x}\\) is shorthand for \\(x_1, x_2, ..., x_k \\)

*Note: Problems with Normal Error Assumption* 

1. Factors in u can have very different distributions in the population. While central limit theorems can still hold, the normal approximation can be poor. 

2. Central limit theorem arguments assume all error affect y in separate additive fashion which has no guarantee of truth. Any breakdown in this assumption will break this assumption 

3. In any given application, normal error assumptions are an empirical matter. In general the assumption will be false if y takes on just a few possible values. 

*Theorem 4.1: Normality of error terms lead to normality of sampling distributions of OLS estimators*

Under the CLM assumptions (MLR Assumptions 1-6) conditional on the sample values of the independent variables 

\\[\hat{\beta_j} \sim N(\beta_j, V(\beta_j)) \\] which implies 

\\[\frac{\hat{\beta_j} - \beta_j}{V(\hat{\beta_j})} \sim N(0,1) \\]

## Testing hypotheses about a single population parameter: The t-test

*Theorem 4.2: t distribution for standard errors* 

Under the CLM assumptions (MLR Assumptions 1-6)

\\[\frac{\hat{\beta_j} - \beta_j}{se(\hat{\beta_j})} \sim t\_{n-k-1} = t\_{df} \\]

where k+1 is the number of paramters in the population model and n-k-1 is the degrees of freedom.

Theorem 2 is important because it allows us to test hypotheses involving \\(\beta_j\\). The test statistic we use is the t-statistic. 

\\[t\_{stat} = t_{\hat{\beta_j}} = \frac{\hat{\beta_j}}{se(\hat{\beta_j})} \\]

## Null Hypothesis Tests

The most common hypothesis test is a two-sided alternative 

\\[H_0: \beta_j = 0 \\]
\\[H_a: \beta_j \neq 0 \\]

We can also test against one sided alternatives where we expect 

\\[H_0: \beta_j \leq 0 \\]
\\[H_a: \beta_j > 0 \\]

or the reverse. 

A p-value is the probability of observing a test statistic, in this case a t-statistic, as extreme as the one we observed given that the null hypothesis is true. 

## Testing Hypotheses about a single linear combination of Parameters 

Our t-statistic keeps the same general format but changes slightly. 

\\[t = \frac{\hat{\beta_j} - \hat{\beta_k}}{se(\hat{\beta_j} - \hat{\beta_k})}\\]

where

\\[se(\hat{\beta_j} - \hat{\beta_k}) = \sqrt{se(\hat{\beta_j})^2 + se(\hat{\beta_k})^2 + 2Cov(\hat{\beta_j},\hat{\beta_k})}\\]

Some guidelines for discussing signficance of a variable in a MLR model 

1. Check for statistical significance. If yes, discuss the magnitude of the coefficient to get an idea of its practical importance. 

2. If variable isn't significant, look at whether the variable has the expected effect and if it is practically large. 

## Confidence Intervals 

A confidence interval is if random samples were obtained infinitely many times with \\(\tilde{\beta_j}, \hat{\beta_j}\\) computed each time then the (unknown) \\(\beta_j\\) population value would lie in the interval \\((\tilde{\beta_j}, \hat{\beta_j})\\) for 95% of the samples. 

Under the CLM assumptions, a confidence interval is 

\\[\hat{\beta_j} \pm \alpha se(\hat{\beta_j}) \\]

where \\(\alpha\\) is a critical value

## Testing Multiple Linear Restrictions: The F-test 

Often we want to test whether a group of variables have no effect on the dependenet variables. 

The null hypothesis is that a set of variables has no effect on y, once another set of variables have been controlled. 

In contest to hypothesis testing:

 - The restricted model is the model without hte groups of variables we are testing 
 - The unrestricted model is the model of all the parameters

For the general case: 

 - The unrestricted model with k independent variables \\(y = \beta_0 + \beta_1x_1 +...+ \beta_k x_k + u\\)

 - The null hypothesis is \\(H_0: \beta\_{k-q-1} = 0,..., \beta_k = 0\\) which puts q exclusion restrictions on the model 

The F-statistic is: 

\\[F = \frac{\frac{SSR_r - SSR\_{ur}}{q}}{\frac{SSR\_{ur}}{n-k-1}} \\]

where \\(SSR_r\\) is the sum of squared residuals from the restricted model and \\(SSR\_{ur} \\) is the sum of squared residuals from the unrestricted model and q is the numerator degrees of freedom which is the degrees of freedom in the restricted model minus the degrees of freedom in the unrestricted model. 

The F-statistic is always non-negative. If \\(H_0\\) is rejected then we say that \\(x\_{k-q+1},...x_k \\) are jointly statistically significant. If we fail to reject then the variables are jointly statistically insignificant. 

## The R-Squared Form of the F-statistic

Using the fact that \\(SSR_r = SST(1-R_r^2) \\) and \\(SSR\_{ur} = SST(1 - R\_{ur}^2) \\) we can substitute in to the F-statistic to get 

\\[F = \frac{\frac{R\_{ur}^2 - R_r^2}{q}}{\frac{1-R\_{ur}^2}{n-k-1}} \\]

This is often more convenient for testing exclusion restrictions in models but cannot test all linear restrictions. 



