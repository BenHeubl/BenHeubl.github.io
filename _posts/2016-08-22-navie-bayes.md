---
title: Building a model to spot how unique Hillary Clinton really is
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - machine learning
tags:
  - update
---

**How unique is Hillary Clinton's style? What does her speeches tell us about her uniqueness? In this post I build several Naive Bayes models, trained them on Hillary's 2016 campaing speeches and applied them on other speeches, tweets and text corpuses. The results is truly interesing.**

![alt text](/images/naive/header.jpg)

I sometimes have a hard time to find applications for machine learning in a data journalism context. While models can predict, there sometimes is simply not a great deal of use-cases that could tell a reader something new or something existing about a current affair. I now have been looking at text analysis for while. Am convinced that it could server as one of the most interesting data sources for data journalists. The big question is how and why.

# Naive Bayes for text analysis

Here, I make a small start to use text apply machine learning and text analysis my a judgement on Hillary Clinton's uniqueness. By using her 2016 campaign speeches from Clinton's campaign website, by mixing it up with speeches from other US presidents (including some of her husband's speeches) from this source, and by training a fairly simple Naive Bayes model by applying a "basket of words" methodology, we test on how good the model performs. Initially i though, if the model performs badly, then it is either my fault of wrongly calibrating the model, the data it was trained on, or - something i have great hope for - may be able to provide evidence on Clinton's uniqueness of her speeches. However, i want to you get too excited. The approach has some downsides.

![alt text](/images/naive/clinton_happy.gif)

While the algorythm is based on a simple Naive Baise model that is fast, and very effective, able to deal with noisy and missing data and requires relatively few examples for training and easy to obtain the estimated probability for a prediction, it also relies on an often-faulty assumption of equally important and independent features. It isn't ideal for datasets with many numeric features and estimated probabilities are less reliable than the predicted classes.

I will run you through the process, before showing you evidence on how well the model performed on...:

- Hillary's own speeches from year ago (giving clues about whether and how she may have changed of her style over the years)
- on a mix of speeches by Hillary's husband Bill
- on a set of Trumps speeches (how difficult is it to distinquish Hillary's from Donald's speeches?),
- and on Hillary's recent tweets (can we spot which ones she didnt write?)

# Get the data:

To train a Naive Bayes model, we need data. We fetch it from Hillary's campaign [webstie][5b26ae86].

```r
# Required packages:
library(xml2)
library(rvest)
library(dplyr)
library(tidyr)
############# 1\. Go thru the next200 button we get all the URLs we need:
get_200_link <- function (url) {
  get <- read_html(url)
  link_200 <- get %>%
    html_node("#mw-pages a:nth-child(4)") %>%
    html_attr("href")
  return(link_200)
}
```

Next we clean it.

[5b26ae86]: https://www.hillaryclinton.com/ "link"
