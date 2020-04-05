---
title: The Standard Errors of Persistence Slides
author: alex
date: '2020-04-05'
slug: the-standard-errors-of-persistence-slides
categories: []
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2020-04-05T11:25:10-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

On Friday, I gave a presentation on Morgan Kelly's "The standard errors of persistence" a summary of which is available [here](https://voxeu.org/article/standard-errors-persistence). 

The jist of Kelly's work is that the persistence literature features unusually high t-statistics in part because of severe spatial autocorrelation in the residuals. When one accounts for this issue, he finds that the main persistence variable frequently has lower explanatory power than spatial noise. Furthermore, that persistence variable often strongly predicts spatial noise. While Kelly is not explicit on this point, the nature of that finding calls into question the results of the papers in his sample. In effect, the regressions are just fitting spatial noise rather than a causal effect. 

The slides are below and the link to all the R code used to create the figures and simulation is [here](https://gist.github.com/asteves/a0da514367e6183aa19983a5db178509). 

## Slides {.tabset}

<iframe src="slides/kelly.html#1" width="672" height="400px"></iframe>
