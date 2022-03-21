---
title: Building Up from our Bootstraps
author: R package build
date: '2022-03-21'
slug: []
categories:
  - R
tags:
  - R
subtitle: ''
summary: ''
authors: []
featured: no
draft: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />



## Thinking about Learning Methods

*Note to Readers: If you arrived at this post because of a search query about how to do bootstrapping, especially wild bootstrapping, in R, skip to the Replication example section at the bottom*.

When I first entered grad school, I knew little to nothing about quantitative methods. The methods sequence at Berkeley was daunting to me, and I felt overwhelmed with all of the new skills and techniques that my cohort seemed to be picking up with ease.[^1]

[^1]: Perhaps unsurprisingly, this insecurity manifested itself in a subpar way. A post for another time.

What was intimidating? In general, a [slide](https://www.math.kth.se/matstat/gru/sf2930/papers/wild.bootstrap.pdf) that looked like this:

![Ahhh!!!](images/Screen%20Shot%202022-03-19%20at%207.02.33%20PM.png)

Over time, these slides became less scary to me, and I *think* I know why. As I got a bit better at methods, I started translating them into implementations in code. Very often, the code implementation made more sense to me. Admittedly, such a learning style has downsides. Methods are a language, and language learners are correctly deterred from trying to translate too much between languages. However, I knew very little. When starting out, I *needed* to make those connections so that eventually, I could read and understand those slides without having to resort to a multi-step process. In short, I needed a bootstrap.

In this post, I am going to give a walk-through of the method by explaining how to well, bootstrap. Introduced by Efron (1979)[^2], the bootstrap is a method for estimating the distribution of a test statistic via repeated resampling of data or a model estimated from data.[^3] Unreasonably effective in multiple applications, the bootstrap provides approximations to the distribution of test statistics, confidence intervals, and rejection probabilities. Extended by other authors, the procedure also works for the situation where the original assumption of independent and identically distributed errors does not hold.

[^2]: Efron, Bradley. "Bootstrap methods: another look at the jackknife." In *Breakthroughs in statistics*, pp. 569-593. Springer, New York, NY, 1992.

[^3]: Two fantastic resources, from which much of this post is drawn are Horowitz, J. 2019. "Bootstrap Methods in Econometrics" *Annual Review of Economics 11:1, 193-224* and Davison, A., & Hinkley, D. (1997). *Bootstrap Methods and their Application* (Cambridge Series in Statistical and Probabilistic Mathematics). Cambridge: Cambridge University Press. <doi:10.1017/CBO9780511802843>. I'd be remiss given my methods upbringing if I didn't mention Freedman's *Statistical Models* chapter on the bootstrap at times.

Why would you be interested in knowing any of these points? Simply put, we do not believe that the data we have from any given experiment or study is complete. Furthermore, we know that when we run the experiment or study again, we are liable to get different data each time. In order to infer a relationship from data, we have to do something about this fundamental uncertainty. Efron's initial insight was that there are times when we can use the data we have as a population and simulate replications. Since we already have a guess about the mechanism that creates the data (our model!), why not trust it and use it to generate data that we hypothesize should have the same distribution as the real data. Do that many times, and we get a sampling distribution. From that sampling distribution, we can draw inferences and insights.

Except if you are me learning about this originally, that was not so clear. Here is what I did when I was trying to learn what was going on.

## Code

Strangely enough, I started with code. Partially this is because I knew a little bit more code than I did math.[^4] The other reason is that seeing how the code was written provided intuition as to what the math meant.[^5] Here's the general process for bootstrapping.

[^4]: Actually a lot more, which says how little I knew about the math and statistics because I do not claim to be a great programmer.

[^5]: The first time this really worked for me was when I learned that a "fixed effect" is just a dummy variable in a regression instead of some dark econometrics magic. I only learned that after looking at someone's R code.

1.  Get some data

2.  Get the answer to something you care about. Take that as the statistic of interest. This might be an average, a standard error, or something more complicated with more Greek letters.

3.  Use the model to generate a bunch of new samples by sampling the data you have repeatedly with replacement.

4.  For each one of those samples, compute the same thing you cared about in (2). This is the *sampling distribution* of the bootstrap.

5.  Use the results from (3) and (4) to (usually) take an average or a standard deviation of that sample distribution. That is your bootstrapped estimate.

The textbook case[^6] is bootstrapping the sample mean.

[^6]: Quite literally at Berkeley because our textbook was Freedman, David A. *Statistical models: theory and practice*. cambridge university press, 2009.


```r
set.seed(12)
## Get some data 
N <- 100
data <- rnorm(N, mean = 10, sd = 2)

## Get a statistic 
sample_mean <- mean(data)
sample_mean 
```

```
## [1] 9.937663
```

```r
## Number of Bootstrap replications 
B <- 10000
bootstrap_means <- vector(mode = "logical", 
                          length = B)
## repeatedly sample the data with replacement 
## for each repetition take the sample mean 
for(i in 1:B){
    bootstrap_means[i] <- mean(sample(data, 
                                      size = N, 
                                      replace = TRUE))
}


## Take the average and standard deviations 
sample_mean_bootstrap <- mean(bootstrap_means)
sample_se_bootstrap <- sd(bootstrap_means)

sample_mean_bootstrap
```

```
## [1] 9.935817
```

```r
sample_se_bootstrap
```

```
## [1] 0.1710695
```


```r
ggplot(data = tibble(
    bootstrap_means = bootstrap_means), 
       aes(x = bootstrap_means))+
    geom_histogram()+
    geom_vline(xintercept = mean(bootstrap_means))+
    theme_minimal()+
    xlab("Replicates")+
    ylab("Count")+
    ggtitle("Bootstrap Distribution")
```

![](/img/bootstrap.png)

Note that in this case, the original sample mean and the bootstrapped sample mean are functionally identical. If I had run more replicates, the procedure for the sample mean is unbiased so would be identical in convergence. The bootstrap standard error gives the uncertainty of this estimate for the data.

Why did I think this kind of exercise worked for me? Well, in part, it is because I already knew the answer, so all I had to do was fill in the appropriate steps. It turns out that this idea actually has [some support](https://www.apreshill.com/blog/2020-12-self-assessments/) in theories of learning and some success in [classrooms](https://www.allisonhorst.com/post/share-keys/). In my undergraduate class on [Applied Causal Inference](https://github.com/asteves/BerkeleyAppliedCausalInference), I experimented with it on the final two assignments and saw great improvement from students. My pet theory is that knowing where one needs to go is empowering for learning. Additionally, it becomes easier to take the next steps to determine why something is correct once you know that your solution is correct. That allowed me to turn towards the intuition of the problem.

## Intuition

Let's go back to the results we found. We have an answer to the question of an estimate of the standard error of the sample mean, but there are some open questions from that simulation. For one, why do we draw from the same data with replacement? And why is it that we take a sample exactly the same size? For that matter, is this distribution actually what we want? Our sample isn't all the data on our variable of interest. That's why it is a sample!

The intuition for the first question is that in the simulation, we imagine all of the different ways we could have drawn 50 observations from a set of numbers. We are, in effect, making many new populations. If we did not set replacement equal to true, we'd get the same sample every time, just in different orders. However, that same point implies a restriction on this bootstrap procedure. We have to have a reason to believe that the thing we want to know comes from a sample where the draws of data are independent and identically distributed. If we set replace to FALSE, the data draws would not be independent. They have to be identically distributed because they are assumed to come from our original sample.

The short answer to the third question is no. What we actually want is many draws from the real data generating process. We do not have that. If we did have that, we would not go through any of this trouble to write out a bunch of formulas. We would just say, "It's 7" and move on with our lives.[^7] What we have is the empirical distribution associated with our original sample of data.[^8]

[^7]: True story. When I was in high school, another student in my class did not study for a quiz and just wrote down the number 7 for every answer. Every answer was in fact 7.

[^8]: What we actually get with lots of different combinations of the numbers that make up our sample.

At this point in a textbook or a lecture, the professor usually makes a point that the bootstrap is not perfect and can fail. Our graph of the empirical distribution looks normally distributed, which crops up whenever the underlying process follows the Weak Law of Large Numbers and the Central Limit Theorem. That suggests that our code is tapping into a process that requires our estimator (here, the sample mean) to be approximately linear and normal. If what we were dealing with was not both, then we are not simulating the right process.

Except at this point, we should worry a little bit if we bootstrap something more complicated than a sample mean. A difficulty with a lot of homework assignments and methods learning is what I refer to as the "draw the owl" problem.

![](https://glebbahmutov.com/blog/images/how-to-draw-fp-owl/how-to-draw-an-owl.jpg)

Most problems we see are not as simple as getting the standard error of a sample mean when we claim independent and identically distributed errors. For many important problems, and frankly basically everything in everyday life, the errors are correlated in some way. Most problems political scientists study have correlated error structures. Voting reform efforts in one state in the United States may or may not affect voters in a different state. Nonetheless, every voter in the state with a voting reform is "treated" to that reform. If you have 4,000,000 voters in a state, the number of effective observations you have is still one state. Beyond that, political scientists *love* regression models and, to be more specific, models estimated via ordinary least squares. So do the economists. Does the bootstrap not apply in the cases?

## Getting Wild with the Bootstrap

Surprisingly with a slight modification, we can use the bootstrap when we have a non-constant variance of the error terms (or if one is feeling fancy heteroskedastic errors). The term of art for these bootstrap models is the "wild" bootstrap.[^9]

[^9]: I have no idea why it is called this. The canonical reference is Liu (1988). I am going to do a little bit of "drawing the owl" here in terms of implementation of it.

What do we do in a wild bootstrap? We have some linear model

$$
Y_i = \hat{\beta}\textbf{X}_i + \epsilon_i
$$

where the error term has a variance that is unknown and (potentially) heteroskedastic. The wild bootstrap does the same thing as the regular bootstrap in terms of steps. The difference is how we generate the new data. Instead of generating new data each time in the previous way, we instead hold the covariates fixed from the original data and generate new errors for our bootstrap replicates by first multiplying the residuals by a random variable and then generating new residuals. The covariates are not resampled. We then randomly select new residuals for each data point. This will result in a new outcome value for each observation.

From this point, we generate a bootstrap sample, estimate the linear model using this sample, compute the statistic of interest, obtain the empirical distribution by repeating the process many times, and then use that distribution for our tests.

For reasons that are described in more detail in texts, the new random variable tends to come from a [Rademacher distribution](https://en.wikipedia.org/wiki/Rademacher_distribution).[^10]

[^10]: See Cameron, Gelbach, and Miller (2008).

From a learning perspective, the reason I eventually understood what was going on with the wild bootstrap, and my understanding is still pretty limited, was that I built upon what I had previously known. Karl Polya's brilliant text [How to Solve It](https://en.wikipedia.org/wiki/How_to_Solve_It) mentions that as a problem-solving strategy, a good first step is to simply ask, "Have I seen this before? If I have, is anything different than when I saw it before?"

In many ways, this is no different than my baseball practice when I was little. Start by learning to swing a baseball bat without a ball. Add the ball, but just on a tee. Move the ball from being on the tee to being underhand lobbed by a coach. Move the ball from lobs to slow pitches from \~20 feet away. Move the ball to slow pitches from the mound. Move the ball to regular speed pitches from the mound. Whenever an issue was found, diagnose by running through similar steps, starting with an easier version of the problem.

My learning built on the foundation of being comfortable with an easier problem that I had solved, and then I saw the same type of problem with a new twist. Most of the progress I made came from this type of deliberate practice and from problems that I found that were explicit about slowing adding new steps. As Allen Iverson would say:

![](https://media.giphy.com/media/3oEjI105rmEC22CJFK/giphy.gif)

Practice is not the game. Methods learning is not about immediately applying to research (though that does help), but problems have to ramp up in an intelligent manner. After designing and teaching a methods course for the first time, I found writing good problems to be the hardest and most time-consuming part of course prep.

## Replication Example

I am going to use the data from [Adam Scharpf](https://adamscharpf.weebly.com/) and [Christian Gläßel's](https://christian-glaessel.weebly.com/) article in the American Journal of Political Science [*Why Underachievers Dominate Secret Police Organizations: Evidence from Autocratic Argentina*](https://doi.org/10.7910/DVN/PGFOXW/1SF78X), which I happen to like a great deal. The paper is interested in who serves in secret police forces, and the authors argue that security officials with weak early career performances join secret police to signal their value to regime leaders. Using data from Argentina, they test the effect of Graduation rank on whether an individual served in Battalion 601, which was the most powerful secret police organization. The paper uses logistic regression, but I am going to work with a plain old linear probability model. I refer to their dataset as police, and the data looks like the following.




| bn601| gradrank| cavalry| infantry|    avglit| academage| cohort|
|-----:|--------:|-------:|--------:|---------:|---------:|------:|
|     0| 3.404255|       0|        0| 0.7628371|       214|     77|
|     0| 4.680851|       1|        0| 0.9685043|       230|     77|
|     0| 5.531915|       0|        0| 0.9698494|       212|     77|
|     0| 5.957447|       0|        0| 0.7044107|       216|     77|
|     0| 6.382979|       0|        1| 0.7628371|       215|     77|
|     0| 7.659574|       0|        0| 0.9685043|       239|     77|

I will replicate Model 3 in Table 1 of the AJPS paper using the amazing `fixest` `lmtest` and `sandwich` packages. `fixest` is as the kids would say "stupid good" and blazingly fast. The other two packages are to perform standard error adjustments on the fly.

This model includes variables for whether an officer was a cavalry officer, an infantry officer, the home literacy rate, and cadet age. The standard errors are heteroskedastic and clustered by cohorts. The first model is a replication of the model in the paper. The second is the equivalent OLS model. I will show bootstrapped errors on the latter.


```r
origModel <- feglm(bn601 ~ gradrank + 
                       cavalry + 
                       infantry+
                       avglit + 
                       academage, 
                   data = police, cluster = ~cohort, 
                   family = "logit")
lpmVersion <- feols(bn601 ~ gradrank + cavalry + 
                        infantry+
                        avglit + academage, 
                    data = police, cluster = ~cohort)
```

<table style="NAborder-bottom: 0; width: auto !important; margin-left: auto; margin-right: auto;" class="table">
<caption>Table 1: Replication of Model 3 from Scharpf and Gläßel</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:center;"> Original </th>
   <th style="text-align:center;"> LPM </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:center;"> −5.51345*** </td>
   <td style="text-align:center;"> −0.03580 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (1.53594) </td>
   <td style="text-align:center;"> (0.04841) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gradrank </td>
   <td style="text-align:center;"> 0.01360*** </td>
   <td style="text-align:center;"> 0.00045** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.00380) </td>
   <td style="text-align:center;"> (0.00016) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cavalry </td>
   <td style="text-align:center;"> −0.55128+ </td>
   <td style="text-align:center;"> −0.01347* </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.30813) </td>
   <td style="text-align:center;"> (0.00641) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> infantry </td>
   <td style="text-align:center;"> 0.51680** </td>
   <td style="text-align:center;"> 0.01958* </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.17105) </td>
   <td style="text-align:center;"> (0.00724) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> avglit </td>
   <td style="text-align:center;"> 1.29053+ </td>
   <td style="text-align:center;"> 0.04371+ </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.69447) </td>
   <td style="text-align:center;"> (0.02303) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> academage </td>
   <td style="text-align:center;"> 0.00108 </td>
   <td style="text-align:center;"> 0.00004 </td>
  </tr>
  <tr>
   <td style="text-align:left;box-shadow: 0px 1px">  </td>
   <td style="text-align:center;box-shadow: 0px 1px"> (0.00600) </td>
   <td style="text-align:center;box-shadow: 0px 1px"> (0.00019) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Num.Obs. </td>
   <td style="text-align:center;"> 4216 </td>
   <td style="text-align:center;"> 4216 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.010 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 Adj. </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.008 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 Within </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 Pseudo </td>
   <td style="text-align:center;"> 0.032 </td>
   <td style="text-align:center;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> AIC </td>
   <td style="text-align:center;"> 1272.7 </td>
   <td style="text-align:center;"> −2254.7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BIC </td>
   <td style="text-align:center;"> 1310.8 </td>
   <td style="text-align:center;"> −2216.6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Log.Lik. </td>
   <td style="text-align:center;"> −630.364 </td>
   <td style="text-align:center;"> 1133.346 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Std.Errors </td>
   <td style="text-align:center;"> by: cohort </td>
   <td style="text-align:center;"> by: cohort </td>
  </tr>
</tbody>
<tfoot><tr><td style="padding: 0; " colspan="100%">
<sup></sup> + p &lt; 0.1, * p &lt; 0.05, ** p &lt; 0.01, *** p &lt; 0.001</td></tr></tfoot>
</table>

As can be seen, we can nearly perfectly replicate the model in the paper and the LPM gives the same general answer regarding significance, which is less important here.

Let's now run several different standard errors, using a trick that I got from [Grant McDermott](https://grantmcdermott.com/better-way-adjust-SEs/). The first type is the type reported in the paper, heteroskedastic errors clustered by cohort. The second and third are two different wild bootstraps using the Rademacher and Mammen distributions to draw the variable to modify the residuals.


```r
lpmV <- lm(bn601 ~ gradrank + cavalry + 
                        infantry+
                        avglit + academage, 
                    data = police)
vcovs <- list(
"Heteroskedastic Clustered" = vcovCL(lpmV, 
                                     cluster = ~ cohort),

"BS-wild-cluster" = vcovBS(lpmV, 
                           cluster = ~cohort, 
                           R = 500, 
                           type = "wild-rademacher"),

"BS-wild-cluster-mammen" = vcovBS(lpmV, 
                                  cluster = ~cohort, 
                                  R = 500, 
                                  type = "mammen")
)

models <- lapply(vcovs,function(x) coeftest(lpmV, vcov =x))
```

<table style="NAborder-bottom: 0; width: auto !important; margin-left: auto; margin-right: auto;" class="table">
<caption>Table 2: Scharpf and Gläßel LPM Models with different standard errors</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:center;"> Heteroskedastic Clustered </th>
   <th style="text-align:center;"> BS-wild-cluster </th>
   <th style="text-align:center;"> BS-wild-cluster-mammen </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:center;"> −0.03580 </td>
   <td style="text-align:center;"> −0.03580 </td>
   <td style="text-align:center;"> −0.03580 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.04841) </td>
   <td style="text-align:center;"> (0.04747) </td>
   <td style="text-align:center;"> (0.04732) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gradrank </td>
   <td style="text-align:center;"> 0.00045** </td>
   <td style="text-align:center;"> 0.00045** </td>
   <td style="text-align:center;"> 0.00045** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.00016) </td>
   <td style="text-align:center;"> (0.00016) </td>
   <td style="text-align:center;"> (0.00016) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cavalry </td>
   <td style="text-align:center;"> −0.01347* </td>
   <td style="text-align:center;"> −0.01347* </td>
   <td style="text-align:center;"> −0.01347* </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.00641) </td>
   <td style="text-align:center;"> (0.00680) </td>
   <td style="text-align:center;"> (0.00660) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> infantry </td>
   <td style="text-align:center;"> 0.01958** </td>
   <td style="text-align:center;"> 0.01958** </td>
   <td style="text-align:center;"> 0.01958** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.00724) </td>
   <td style="text-align:center;"> (0.00729) </td>
   <td style="text-align:center;"> (0.00705) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> avglit </td>
   <td style="text-align:center;"> 0.04371+ </td>
   <td style="text-align:center;"> 0.04371+ </td>
   <td style="text-align:center;"> 0.04371+ </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.02303) </td>
   <td style="text-align:center;"> (0.02303) </td>
   <td style="text-align:center;"> (0.02253) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> academage </td>
   <td style="text-align:center;"> 0.00004 </td>
   <td style="text-align:center;"> 0.00004 </td>
   <td style="text-align:center;"> 0.00004 </td>
  </tr>
  <tr>
   <td style="text-align:left;box-shadow: 0px 1px">  </td>
   <td style="text-align:center;box-shadow: 0px 1px"> (0.00019) </td>
   <td style="text-align:center;box-shadow: 0px 1px"> (0.00018) </td>
   <td style="text-align:center;box-shadow: 0px 1px"> (0.00019) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Num.Obs. </td>
   <td style="text-align:center;"> 4216 </td>
   <td style="text-align:center;"> 4216 </td>
   <td style="text-align:center;"> 4216 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> AIC </td>
   <td style="text-align:center;"> −2252.7 </td>
   <td style="text-align:center;"> −2252.7 </td>
   <td style="text-align:center;"> −2252.7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BIC </td>
   <td style="text-align:center;"> −2208.3 </td>
   <td style="text-align:center;"> −2208.3 </td>
   <td style="text-align:center;"> −2208.3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Log.Lik. </td>
   <td style="text-align:center;"> 1133.346 </td>
   <td style="text-align:center;"> 1133.346 </td>
   <td style="text-align:center;"> 1133.346 </td>
  </tr>
</tbody>
<tfoot><tr><td style="padding: 0; " colspan="100%">
<sup></sup> + p &lt; 0.1, * p &lt; 0.05, ** p &lt; 0.01, *** p &lt; 0.001</td></tr></tfoot>
</table>

If you're using Stata, or have a co-author who feels the urge to use R packages that are ports from Stata packages, then [boottest](https://github.com/droodman/boottest) is the package that you want. The Stata Journal article is [here](https://journals.sagepub.com/doi/abs/10.1177/1536867X19830877?journalCode=stja).

## Conclusion 

Methods can be hard, and often make me feel like I know very little about the world. Today, I tend to like that feeling because I have developed a tool or two over time to push forward in my own learning. For those who feel lost now, do not despair. I was where you are now, and frequently still am. What I would strongly encourage is to find your own bootstrapping process for learning.
