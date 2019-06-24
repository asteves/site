---
title: 'Simulating Football Recruiting Evaluations with R '
author: alex
date: '2019-06-24'
slug: simulating-football-recruiting-evaluations-with-r
categories: [R]
tags: []
subtitle: ''
summary: 'R is great for running quick simulations. Using a running example of college football recruit rankings, I show how we can leverage the power of R to see the implication of evaluators of different quality.'
authors: []
lastmod: '2019-06-24T12:39:14-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Introduction 
--------------

A common problem in the world is reporting point estimates without estimates of error. While this issue crops up in many important domains, it is extremely common in college football recruiting. [247 Sports Team Rankings](https://247sports.com/Season/2020-Football/RecruitRankings/?InstitutionGroup=HighSchool) report recruit rankings as single numbers. According to 247, the top recruit in the country has a rating of 98 out of 100.* 

An oft repeated aphorism is [recruiting matters](https://247sports.com/Article/College-football-signing-day-2019-why-recruiting-matters-128646121/). Teams that recruit well win more national championships than teams that do not. A somewhat oft repeated response is that recruiting websites are poor judges of lots of players. 

For the researcher interested in evaluating the problem, there are two crucial pieces of information missing from the recruiting sites. The first is conceptual. Recruiting sites cannot rank and assign values to development in college. A can't miss recruit may arrive at University and struggle to fit into a new environment, get injured, or have a myriad of other problems that prevent them from reaching their potential. There is nothing that recruiting sites can do about that, and to their credit, such sites tend to evaluate on potential. 

The second piece is more interesting. Recruiting sites do not report their level of uncertainty. Furthermore, recruiting websites often go back and reevaluate players after they have signed with a major college football program. This means that some prospects can be evaluated many times, while other prospects at most twice. If a recruit has been evaluated 100s of times, there is a reasonable chance that evaluators can converge to the actual potential of a recruit. Again this will have some error, but the law of large numbers will kick in. 

Suppose instead that our evaluators are terrible but not systematically biased. That is they generate really noisy evaluations. What's the difference between their evaluations and the actual potential? To find out, we can simulate such a problem with R. The following is one extremely simplified way to do just that. 


Functions 
--------------

We will make use of the tidyverse and the very cool ggridges package for plotting. 


```r
library(tidyverse)
library(ggridges)
```

The first function that we'll write is one to get recruiter evaluations. This function bins evaluators into three categories: good, ok, and terrible. If you follow college recruiting websites, those categories usually describe 247, Rivals, and ESPN respectively. For each kind of evaluator, we will have them look at prospects (for whom we know the actual talent), and generate a ranking from a normal distribution. Every recruiter will on average as n goes to infinity converge to the same mean, but each has different amount of noise. 


```r
evaluate_recruits <- function(input, evaluator = "terrible") {
    data <- rep(NA_integer_, length(input))
    if (evaluator == "terrible") {
        for (i in seq_along(input)) {
            # Inputs are from (0,1] so a sd of .08 is real noisy
            data[i] <- rnorm(1, mean = input[i], sd = .08)
        }
        return(data)
    }
    if (evaluator == "ok") {
        # sd is half as bad as the bad evaluator
        for (i in seq_along(input)) {
            data[i] <- rnorm(1, mean = input[i], sd = .04)
        }
        return(data)
    }
    if (evaluator == "good") {
        # sd is 1/3 as bad as the bad evaluator
        for (i in seq_along(input)) {
            data[i] <- rnorm(1, mean = input[i], sd = .02)
        }
        return(data)
    }

}
```

The following plotting function is just a wrapper around ggridges. 


```r
make_plot <- function(df, N){
    #  Generate title
    title <-paste("Simulated Evaluations with ", N, " replications", sep = "")

    # Make ridge plot
    plot <- ggplot(df,
                   aes(
                       x = value,
                       y = evaluator,
                       group = evaluator,
                       fill = evaluator
                   )) +
        geom_density_ridges(
            jittered_points = TRUE,
            point_shape = "|",
            point_size = 5,
            position = position_points_jitter(height = 0, width = .01),
            alpha = .3
        ) +
        theme_ridges() +
        ylab("") +
        stat_density_ridges(quantile_lines = TRUE, quantiles = 2) +
        xlab("") +
        ggtitle(label = title)
    print(plot)
}
```

Next we write a function to simulate different number of evaluations. R has a great function, replicate() that allows us to repeat a function an arbitrary amount of times. We will generate three different classes of evaluations for the same recruits and then pass them to make_plot()


```r
sim_evals <- function(N, actual) {

    # Draw replicates
    terrible <- rowMeans(replicate(N, evaluate_recruits(actual, "terrible")))
    ok <- rowMeans(replicate(N, evaluate_recruits(actual, "ok")))
    good <-
        rowMeans(replicate(N, evaluate_recruits(actual, "good")))

    # Join as a data frame and put into long format
    df <- data.frame(terrible, ok, good, actual) %>%
        gather(evaluator, value)
    make_plot(df, N)
}
```


Simulating Examples 
--------------

Let's see what kind of distributions we get of the same talent that coincidentally happen to be the current point estimates for the University of Minnesota's 2019 recruiting class from the 247 Composite. Two key points come into play. First, if every recruit is watched a lot then each kind of evaluator will get close to the overall class mean and have similar distributions. Second, as the number of evaluations drop we see large variance between recruiter types even as the point estimates for the class means are relatively stable. 


```r
set.seed(6242019)
N <- c(1, 10, 100, 1000)

actual <- round(c(.8789,.8668,.8577,.8552,
            .8552,.8539,.8539,.8532,
            .8506,.8499,.8471,.8466,
            .8465,.8465,.8452,.8431,
            .8366,.8316,.8299,.8299),2)

for(i in seq_along(N)){
    suppressMessages(sim_evals(N[i], actual))
}
```

<img src="/post/2019-06-24-simulating-football-recruiting-evaluations-with-r_files/figure-html/unnamed-chunk-5-1.png" width="672" /><img src="/post/2019-06-24-simulating-football-recruiting-evaluations-with-r_files/figure-html/unnamed-chunk-5-2.png" width="672" /><img src="/post/2019-06-24-simulating-football-recruiting-evaluations-with-r_files/figure-html/unnamed-chunk-5-3.png" width="672" /><img src="/post/2019-06-24-simulating-football-recruiting-evaluations-with-r_files/figure-html/unnamed-chunk-5-4.png" width="672" />

One conclusion that I draw from this simulation is that prattling on about point estimates when forecasting is a fool's errands. Another is that spending lots of time following college football recruiting is strange. A third is that it is extremely easy to write simulations in R. 


*Technically out of 110, but since recruits who receive a higher grade than 100 are once in a generation players 100 is a decent cut point. 
