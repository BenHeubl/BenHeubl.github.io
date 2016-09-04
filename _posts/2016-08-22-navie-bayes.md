---
title: Naive Bayes for text analysis
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - machine learning
tags:
  - update
---

![alt text](/images/navie/header.png)

# Naive Bayes for text analysis

**intro.**

bla:

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
