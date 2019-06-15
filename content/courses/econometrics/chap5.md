---
title: "OLS Asymptotics"
linktitle: OLS Asymptotics
toc: true
type: docs 
date: "2019-06-14T00:00:00+01:00"
draft: false 
menu:
  econometrics:
    parent: Econometrics
    weight: 4

weight: 4
---

We know under certain assumptions that OLS estimators are unbiased, but unbiasedness cannot always be achieved for an estimator. Another property that we are interested in is whether an estimator is *consistent*. 

*Theorem 5.1: OLS is a consistent estimator*

Under MLR Assumptions 1-4, the OLS estimator \\(\hat{\beta_j} \\) is consistent for \\(\beta_j \forall \\ j \in 1,2,...,k\\). 

Informally, as n tends to infinitythe distribution of \\(\hat{\beta_j} \\) collapses to the single point \\(\beta_j \\)

We can add an assumption MLR 4': 

\\[E(u) = 0,Cov(x_j, u) = 0 \forall j \in 1,2,...,k \\]

This is a weaker assumption than MLR 4. MLR 4' requires only that \\(x_j \\) is uncorrelated with u and that u has zero mean in the population. Indeed MLR 4 implies MLR 4'. 

We use MLR4 as an assumption because OLS is biased but consistent under MLR 4' if \\(E[u| x_1, ..., x_k]\\) depends on any of the \\(x_j \\). Second, if MLR 4 holds, then we have properly modeled the population regression function. 

## Deriving Inconsistency of OLS 

Correlation between u and any of the \\(x_1, ..., x_k]\\) generally causes all of the OLS estimators to be inconsistent. **If the error is correlated with any of the independent variables then OLS is biased and inconsistent.**

There is an asymptotic analogue to Omitted Variable Bias. Suppose the model \\(y = \beta_0 + \beta_1x_1 + \beta_2x_2 + v \\) satisfies MLR assumptions 1-4. If we omit \\(x_2\\) then: 

\\[plim\tilde{\beta_1} = \beta_1 + \beta_2\delta_1 \\]

\\[plim\tilde{\beta_1} = \beta_1 + \beta_2\frac{Cov(x_1,x_2)}{V(x_1)} \\]

If the covariance term is zero then the estimator is still consistent. Otherwise, the inconsistency takes on the same sign as the covariance term. 

## Asymptotic Normality and Large Sample Inference

In cases where the \\(y_i\\) do not follow normal distributions we can still get asymptotic normality. 

*Theorem 5.2: Asymptotic Normality*

Under MLR Assumptions 1-5

 1. \\(\sqrt{n}(\hat{\beta_j} - \beta_j) \xrightarrow{a} N(0, \frac{\sigma^2}{a_j^2}\\) where \\(a_j^2\\) is the asymptotic variance of \\(\sqrt{n}(\hat{\beta_j} - \beta_j)\\). For the slope coefficients \\(a\_j^2 = plim(\frac{1}{n} \sum\_{i=1}^n \hat{r\_{ij}^2})\\) where the \\(r\_{ij}\\) are the residuals from regressing \\(x_j\\) on the other independent variables. 

 2. \\(\hat{\sigma^2} is a consistent estimator of \sigma^2)

 3. For each j 

 \\[\frac{\hat{\beta_j} - \beta_j}{sd(\hat{\beta_j})} \xrightarrow{a} N(0,1)\\] which we cannot compute from data and

\\[\frac{\hat{\beta_j} - \beta_j}{se(\hat{\beta_j})} \xrightarrow{a} N(0,1)\\] which we can compute from data. 


This theorem does not require MLR 6 from the list of required assupmtions. What this theorem says is that regardless of the population distribution of u, the OLS estimators when properly standardized have approximate standard normal distributions. 

Further, 

\\[\frac{\hat{\beta_j} - \beta_j}{se(\hat{\beta_j})} \xrightarrow{a} t\_{df}\\]

because \\(t\_{df} \\) approaches \\( N(0,1)\\) as the degrees of freedom gets large so we can carry out t-tests and confidence intervals in the same way as the CLM assumptions. 

Recall that \\(\widehat{V(\hat{\beta_j})} = \frac{\hat{\sigma^2}}{SST_j(1-R_j^2)} \\) where \\(SST_j \\) is the total sum of squares of \\(x_j\\) in the sample and \\(R_j^2 \\) is the R-squared from regressing \\(x_j\\) on all other independent variables. As the sample size increases: 

 - \\(\hat{\sigma^2} \xrightarrow{d} \sigma^2\\)
 - \\(R_j^2 \xrightarrow{d} c\\) which is some number between 0 and 1 
 - The sample variance \\(\frac{SST_j}{n} \xrightarrow{d} V(x_j)\\)

These imply that \\(\widehat{V(\hat{\beta_j})} \\) shrinks to 0 at the rate of \\(\frac{1}{n}\\) and \\(se(\hat{\beta_j}) = \frac{c_j}{\sqrt{n}} \\) where \\(c_j = \frac{\sigma}{\sigma\sqrt{1 - \rho_j^2}}\\). 

This last equation is an approximation. A good rule of thumb is that standard eerrors can be expected to shrink at a rate that is the inverse of the square root of the sample size. 

## Asymptotic Efficiency of OLS 

*Theorem 5.3*

Under Gauss-Markov assumptions, let \\(\tilde{\beta_j} \\) denote estimators that solve the equation

\\[\sum\_{i=1}^n g_j(\textbf{x}_i)(y_i - \tilde{\beta_0}-\tilde{\beta_1}x\_{i1} - ... - \tilde{\beta_k}x\_{ik}) = 0 \\] 

Let \\(\hat{\beta_j} \\) denote the OLS estimators. The OLS estimators have the smallest asymptotic variance. 

\\[AVar(\sqrt{n}(\hat{\beta_j} - \beta_j)) \leq AVar(\sqrt{n}(\tilde{\beta_j} - \beta_j))\\]