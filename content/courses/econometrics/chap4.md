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


