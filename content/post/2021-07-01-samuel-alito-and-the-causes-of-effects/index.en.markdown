---
title: Samuel Alito and the Causes of Effects
author: R package build
date: '2021-07-01'
slug: []
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-07-01T14:48:45-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In *Brnovich et al. v. Democratic National Committee et al.*, the Supreme Court decided once again to gut the enforcement ability of the Voting Rights Act and provide cover for hostile laws designed to make voting meaningfully difficult for non-white people. Justice Samuel Alito, writing for the majority and crafting legal fairy dust to justify it, decided to discuss causal inference. He writes 

> The size of any disparities in a rule's impact on members of different racial or ethnic groups is also an important factor consider. Small disparities are less likely than large ones to indicate that a system is not equally open. To the extent that minority and non-minority groups differ with respect to employment, wealth, and education, even neutral regulations, no matter how crafted may well result in some predictable disparities in rates of voting and non-compliance with voting rules. But the mere fact there is some disparity in impact does not necessarily mean that a system is not equally open or that it does not give everyone an equal opportunity to vote. The size of any disparity matters. And in assessing the size of any disparity, a meaningful comparison is essential. What are at bottom very small differences should not be artifically magnified. 

This is not wrong [^1], and indeed is part of a set of challenges when considering the policy effect. Holland (1986) refers to this as the difference between the causes of effects and the effects of causes. The latter is conceptually far more straightforward because we **know** the cause that we are testing. For example, if I am testing a drug's efficacy of preventing headaches at say 500mg, I can conduct an experiment where I give a set of subjects 500mg of the drug. The former is akin to asking the query, "My headache is passed. Is it because I took 500mg of that drug" (see Dawid 2000). 

After that throat clearing, Alito then proceeds to discuss effect sizes, writing: 

> Next, the racial disparity in burdens allegedly caused by the out-of-precinct policy is small in absolute terms. The District Court accepted the plaintiffs' evidence that, of the Arizona counties that reported out-of-precinct ballots in the 2016 general election, a little over 1% of Hispanic voters, 1% of African-American voters, and 1% of Native American voters who voted on election day cast an out-of-precinct ballot. For non-minority voters, the rate was around 0.5%. A policy that appears to work for 98% or more of voters to whom it applies[^2]--minority and non-minority alike--is unlikely to render a system unequally open. 

Here we have an outcome about the number of out-of-precinct ballots being cast. This is an effect, and we want to reason about the cause. Can we tell whether this is actually a small difference based on this information? 

In general, no, in part because it is easy to develop a model for which this happens purely by chance. Suppose we have a continuum of identical voters. Each voter independently has a 1% chance of voting at the wrong polling place. In expectation[^3], we expect one voter to vote in the wrong precinct, with a standard deviation of about one voter. 

We can lazily simulate this with R as follows:


```r
set.seed(42)
incorrect_votes <- vector(length = 10000)
for(i in 1:10000){
    incorrect_votes[i]<- sum(rbinom(100, 1, prob = 0.01))
}

round(mean(incorrect_votes))
```

```
## [1] 1
```

```r
round(sd(incorrect_votes))
```

```
## [1] 1
```

In our simulation of 10,000 runs, 775 of them turn up a situation where three or more voters vote incorrectly. Our extremely simple model captures a situation where we could see disparities. Our simulation has been constructed by the assumption that vote decisions are independent of race or ethnicity. Nevertheless, we come up with 775 runs for which we could find a disparity if we added information on racial or ethnic identity. 

But do we buy this model? 

I hope not, though it's challenging to read Alito's opinion and imagine that he has a different model in mind. Our model is divorced from multiple aspects of the world. The model assumes that incorrect voting rates are equivalent across our two groups and that there are no other reasons besides individual voter error. By construction, we say every voter is the same, a way to assume away any differences based on race or ethnicity. We have no reason to believe this is true in Arizona. Our outcome, voting rates, is fraught with selection issues. All we see are the number of people who cast a ballot, either in their correct precinct or not. We do not observe all the eligible voters who might have voted but chose not to because of challenges in getting to their polling place.

What might be some reasons why minority and non-minority voters have different rates of exposure to out-of-precinct voting. For one, people in Arizona do not live in places at random, and Arizona politicians move polling places that service minority populations predominantly far more often.[^4] In Arizona, minority voters also have to travel farther to polling places. The choice of where to put precinct locations makes minority voters disproportionately likely to be assigned to polling places other than the ones closest to where they live. Thus, comparing voting rates in the wrong precinct between groups of voters does not compare like for like. If anything, it's rather impressive that groups that have to do more to vote manage to do it correctly in the aggregate just about as often as groups that have to do less. 

Absent some other information, a rigorous reading of causal inference makes it challenging to evaluate between the two. The overall implication is that we should never start at the effects and reason back to the causes. Far better to start with the cause and consider the effect. For example, what is the effect of disproportionately moving precincts in heavily minority populated areas? 

Since we, and the Supreme Court, do not have a clean research design to evaluate, our analyses will thus introduce some bias. However, every claim about bias is a claim about a belief of what that bias is [(Little and Pepinsky 2020)](https://www.journals.uchicago.edu/doi/10.1086/710088). What should our belief be about the direction of that bias? It seems pretty clear to this author that in a society where: (1) most residential concentration is based on historically discriminatory policies, (2) since it became illegal to prevent minorities voting, groups have spent considerable effort crafting policies to do precisely that but sound facially neutral (3) one party consistently screams about the dangers of non-existent fraud and frequently conflates fraudulent voters with the color of a person's skin, and (4) that same party believes that every election they do not win must be stolen, I see no reason to believe as a good Bayesian that Arizona's restrictions are purely neutral. 

[^1]: Strictly speaking, the claim that small disparities are less likely than large ones to indicate that a system is not equally open is impossible to verify without knowledge of the data generating process for the differences we observe.

[^2]: Note that using percentages here masks absolute total since there are almost twice as many non-minority voters are white voters.

[^3]: Each voter has a success or failure, which makes it a Bernoulli random variable. The distribution of the number of successes in a sequence of Bernoulli trials is the Binomial distribution. The expectation of this sequence is the number of trials multiplied by the probability of success (np). The standard deviation is the square root of np(1-p).

[^4]: 30% more according to Kagan's dissent.
