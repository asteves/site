---
title: 'Learning R: Dealing Playcards'
author: alex
date: '2019-06-02'
slug: learning-r-dealing-playcards
categories:
  - R
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2019-06-02T17:16:48-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Introduction 
--------------

There are lots of benefits to learning a computer language, even if only learning enough to make yourself dangerous to your own computer. In this post, I'll provide a purposely dumb way to shuffle playing cards and deal hands to different players. In doing so, I will demonstrate a variety of basic operations in the R language. 


```r
# This is a comment. We write comments for future programmers 
# to be able to read and understand our code. 
# The most common future programmer to read your code is you 
# in the future. 
# Be nice to future you. 

# Comments in R start with a #. 
# R will ignore everything after a # on the same line

# These next lines set a random number seed 
# and then sample 10 numbers from 1 to 100 with replacement. 
#As a result, we might get the same number more than once. 

set.seed(106108)
sample(x = 1:100, size = 10,replace = TRUE, prob = NULL)
```

```
##  [1] 83 18 20  2 69 93 20 93 68 33
```

sample() is a function, or a tool to get stuff done. We can tell something is a function because it has () immediately following it. Note that functions are case sensitive. R does not know what Sample() means. sample comes with the base R distribution, which means you don't have to write it yourself. sample takes four arguments. When googling you may also see arguments referred to as parameters. They are the information the function needs to work.

The four arguments are x, which is a vector of elements. In this case x is every number from 1 to 100. The second argument is size, which is the number of elements we want. In our case, that is 10 numbers. The third argument is replace, which determines whether we want to sample with or without replacement. The final argument is prob, which we ignore by setting it to NULL.

Dealing Cards 
--------------

Sampling with replacement comes up a lot in applications with simple random sampling, but for dealing cards we want to sample without replacement. When we deal a playing card, we do not deal that playing card again to another player. 

Let's use R to assign playing cards at random for practice. First, we need to create a vector of playing cards. This will be a string vector, which means that each element will be a piece of text instead of just a number. To R, "10" is different than 10.


```r
# First, create a vector of cards and assign it the variable cards 
cardTypes <- c(2:10, "J","Q","K", "A")

# R now knows about the existence of cardTypes. 
# We will print out the vector to confirm 
print(cardTypes)
```

```
##  [1] "2"  "3"  "4"  "5"  "6"  "7"  "8"  "9"  "10" "J"  "Q"  "K"  "A"
```

Next, we need to attach the suit to each card. In R, one way to combine text is to use paste(). Note that paste() has parentheses, so it is a function. To look up what arguments we need we type ?paste into the console.


```r
# Attach suits to each card 
hearts = paste(cardTypes, "H", sep = "")
spades = paste(cardTypes, "S", sep = "")
clubs = paste(cardTypes, "C", sep = "")
diamonds = paste(cardTypes, "D", sep = "")

# combine our playing cards into one vector  
playingCards = c(hearts, spades, clubs, diamonds)
print(playingCards)
```

```
##  [1] "2H"  "3H"  "4H"  "5H"  "6H"  "7H"  "8H"  "9H"  "10H" "JH"  "QH" 
## [12] "KH"  "AH"  "2S"  "3S"  "4S"  "5S"  "6S"  "7S"  "8S"  "9S"  "10S"
## [23] "JS"  "QS"  "KS"  "AS"  "2C"  "3C"  "4C"  "5C"  "6C"  "7C"  "8C" 
## [34] "9C"  "10C" "JC"  "QC"  "KC"  "AC"  "2D"  "3D"  "4D"  "5D"  "6D" 
## [45] "7D"  "8D"  "9D"  "10D" "JD"  "QD"  "KD"  "AD"
```


```r
# Confirm that we have 35 elements
print(length(playingCards))
```

```
## [1] 52
```


Now that we have our playing cards loaded into R, we can use sample like before to assign them along with a new function called split(). split() requires two arguments, a vector and the groups. We can always see what arguments split() takes with ?split. Note that the help page has more than two arguments, but we only need to provide two because the others have sensible defaults.


```r
# One of the cool things about functions in R is 
# that we can use them inside other functions. 
#Our tools work on other tools. 
#The groups argument is called f. You can see on the help page why. 

# Here we are hardcoding that we have seven players. If you were to 
# rewrite this code in the future, this is one place you might want to 
# revise and write your own function
hands = split(x = sample(playingCards, 35, replace = F), f = c("1","2","3","4","5","6","7"))
```

Our new object groups looks different than our other objects. That is because it is a "list" instead of a vector. A list is a collection of vectors that can have different types. We will learn about lists more as we move through the semester.

Here are the hands we dealt


```r
print(hands)
```

```
## $`1`
## [1] "QD" "4C" "3H" "6C" "QS"
## 
## $`2`
## [1] "3D" "QH" "4H" "2S" "8H"
## 
## $`3`
## [1] "7C" "KS" "KH" "2H" "4S"
## 
## $`4`
## [1] "9D" "KC" "9S" "9H" "4D"
## 
## $`5`
## [1] "7H" "JD" "8S" "9C" "3S"
## 
## $`6`
## [1] "8C" "6H" "7D" "AS" "5C"
## 
## $`7`
## [1] "AD"  "7S"  "AC"  "5H"  "10C"
```

Here is all of our code in one place.


```r
# Set a random number seed for reproducibility 
set.seed(106108)

# Create a vector of to represent card types and assign it the variable cards 
cardTypes <- c(2:10, "J","Q","K", "A")

# Attach suits to each card 
hearts = paste(cardTypes, "H", sep = "")
spades = paste(cardTypes, "S", sep = "")
clubs = paste(cardTypes, "C", sep = "")
diamonds = paste(cardTypes, "D", sep = "")

# combine our playing cards into one vector along with the two jokers 
playingCards = c(hearts, spades, clubs, diamonds)

# Deal hands
hands = split(x = sample(playingCards, 35, replace = F), f = c(1,2,3,4,5,6,7))
print(hands)
```

```
## $`1`
## [1] "KC" "JD" "4D" "9D" "9S"
## 
## $`2`
## [1] "7S" "8D" "2D" "KD" "7D"
## 
## $`3`
## [1] "6S" "5H" "6D" "JS" "6C"
## 
## $`4`
## [1] "8S"  "8C"  "QH"  "10D" "5C" 
## 
## $`5`
## [1] "3H" "3D" "KS" "4H" "2S"
## 
## $`6`
## [1] "6H" "7C" "AD" "5S" "2H"
## 
## $`7`
## [1] "4C" "7H" "JC" "KH" "9H"
```
