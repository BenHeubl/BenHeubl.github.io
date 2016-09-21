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

Here, I make a small start to use text apply machine learning and text analysis my a judgement on Hillary Clinton's uniqueness. By using her 2016 campaign speeches from Clinton's campaign website, by mixing it up with speeches from other US presidents (including some of her husband's speeches) from this source, and by training a fairly simple Naive Bayes model by applying a "basket of words" methodology, we test on how good the model performs.

![alt text](/images/naive/clinton_happy.gif)

My plan for this post was that if the model performs badly, then i could either blame myself on wrongly calibrating the algorithm, the or the text data it was trained upon, or - which I have my eyes on - be able to provide evidence Hillary Clinton's speeches are unique enough to classify her speeches correctly. However, i want to you get too excited. The approach has some catches. As we only apply a bag of words methodology, the most frequent words impact the search for probability of the classifier. So the time, the speeche was deliverd is an imporant criteria, and was not taken into account her. Naive Bayes classification (NB) on top, has some additional drawbacks.

While the Naive Bayes classifier is said to be fast, and very effective, able to deal with noisy and missing data and requests relatively few examples for training (it is easy to also obtain the estimated probability for a prediction), it relies on an often-faulty assumption of equally important and independent features. NB isn't ideal for datasets with many numeric features and estimated probabilities are less reliable than the predicted classes.

![alt text](/images/naive/nb.jpg)

I will you run through the process how prepare text data and how to classify Hillary's speeches and text. For this, we will be looking at how well NB can perform on text classification for the following:

- find her speeches in a pile mixed up with her husband's Bill (Is she unique enough for the algorithm to spot hers?)
- Hillary's own speeches from the time when she was Secretary of State (giving clues about whether she might have "changed her style" over the past years)
- and on Hillary's recent tweets (can we spot which ones she may have not written?)

# Get the data:

To train a Naive Bayes model, we need text data. We fetch it from Hillary's campaign [webstie][5b26ae86].

```r
# Clinton 2016 speeches from Hillaryclinton.com:
library(xml2)
library(rvest)
library(dplyr)
library(tidyr)

url1 <- "https://www.hillaryclinton.com/speeches/page/"
get_linkt<- function (ur) {
  red_t <- read_html(ur)
  speech <- red_t %>%
    html_nodes(".o-post-no-img") %>%
    html_attr("href")
  return(paste0("https://www.hillaryclinton.com", speech, sep = ""))
}
df_clinton_2016 <- NULL
for (t in 1:10) {
  linkt <- paste0(url1, t, "/", sep = "")
  print(linkt)
  df_clinton_2016 <- rbind(df_clinton_2016, as.data.frame(get_linkt(linkt)))
}

getspe <- function (urs) {
  red_p <- read_html(urs)
  speech1 <- red_p %>%
    html_nodes(".s-wysiwyg") %>%
    html_text()
  as.character(speech1)
  wann <- red_p %>%
    html_node("time") %>%
    html_text()
  as.character(wann)
  dataframs <- cbind(as.data.frame(speech1), as.data.frame(wann))
  return(dataframs)
}

fin_2016 <- NULL
for (p in 1:nrow(df_clinton_2016)) {
  tryCatch({
    linkp <- df_clinton_2016[p, 1]
    print(linkp)
    fin_2016 <- rbind(fin_2016, as.data.frame(getspe(as.character(linkp))))
  }, error=function(e){cat("ERROR :",conditionMessage(e), "\n")})
}
get_speech_2016 <- function (u) {
    red_s <- read_html(u)
    speech <- red_s %>%
      html_nodes() %>%
      html_text()
    wann <- red_s %>%
      html_nodes() %>%
      html_text()
    wo <- red_s %>%
      html_nodes() %>%
      html_text()
    bingins <- cbind(as.data.frame(speech), as.data.frame(wann), as.data.frame(wo))
}
```

Alternatively, just download the data here.

# Is Hillary only a new Bill?

How different is her speech content from Bill Clinton. How unique are the messages she is sending in her 2016 campaign speeches. First we will clean and then train and test the NB on a dataset that contains both Hillary's and Bill's speeches.

## Text data fetching:

For each part of this post, we will both train the model, and then predict, resulting in a judgement how well the model has worked on the given data. For this we need our data to be cleaned. For demonstration purposes, we will do it here once, but skip over this step later on.

First off, I used this [website](http://millercenter.org/president/speeches) to scrape some of Bill Clinton's speeches (they might not be the best in the world - for the reasons explained earlier, but they can serve our purpose to run the classification model. We will mix them up with Hillary's 2016 speeches:

```r

ge_links <- function (x) {
  t <- read_html(x)
  linky <- t %>%
    html_nodes(".title a") %>%
    html_attr("href")
  return(paste0("http://millercenter.org", linky, sep = ""))
}

url_pre <- "http://millercenter.org/president/speeches"

df_pres <- NULL
df_pres <- rbind(df_pres, as.data.frame(paste0("", ge_links(url_pre), sep = "")))

######### change column with column id
names(df_pres)[1]<-"link"

View(df_pres)

get_speech_pres <- function (x) {
  t <- read_html(x)
  speech <- t %>%
    html_node("#transcript") %>%
    html_text()
  return(as.data.frame(speech))
}

get_speech_pres("http://millercenter.org/president/washington/speeches/speech-3459")

df_speeches2 <- NULL
for (y in 90:128) {
  print(y)
  tryCatch({
    link_y = as.character(df_pres[y, 1])
    df_speeches2 <- rbind(df_speeches2, (get_speech_pres(link_y)))
  }, error=function(e){cat("ERROR :",conditionMessage(e), "\n")})
}

# Build a common structure - type and text as columns:
Bills <- df_speeches2  # Bill's speeches
Bills <- Bills %>%
  mutate(type = "Bill") %>%
  mutate(text = speech) %>%
  select(-speech)

Hillary_2016 <-  fin_2016 # Hillary's speeches
Hillary_2016 <- Hillary_2016 %>% select(speech1) %>%
  mutate(type = "Hillary") %>%
  mutate(text = speech1) %>%
  select(-speech1)

# rbind them (concatenate them):
bill_hillary <- rbind(Hillary_2016, Bills)

# Mix them up - apply a random sampling function
nrow(final_clintons) # [1] 135
bill_hillary_random <- sample_n(bill_hillary, 135) # sample_n, a great dplyr function for random sampling
```

# Clean the data:

Here we go, we have the data we need in place. To help with data cleaning, the text processing package 'tm' by Ingo Feinerer is of great help. To start things off, we need a corpus, a simple collection of text documents. We use the VCorpus() function in the tm package after we convert the features in the column type into the data format of type factor.

```r
library(NLP)
library(tm)

# Convert type column into as.factor()
bill_hillary_random <- bill_hillary_random %>%
  mutate(type = factor(type))

# Check distributions:
prop.table(table(bill_hillary_random$type))

# 28% are Bills speeches, the rest are Hillary's
Bill   Hillary
0.2888889 0.7111111
```

We use the VectorSource() reader function to create a source object from the existing hil_bill$text vector, which can then be converted via VCorpus().

```r
hil_bill_corpus <- VCorpus(VectorSource(bill_hillary_random$text))

# Check class:
class(hil_bill_corpus)  # [1] "VCorpus" "Corpus"
```

Think of the tm's coprus object as a list. We can use the list operations to select documents in it and use inspect() with list operators to access text corpus elements.

Next we create a document-term matrix via tm. A [document-term matrix](https://en.wikipedia.org/wiki/Document-term_matrix) or term-document matrix is a mathematical matrix that describes the frequency of terms that occur in a collection of documents. In a document-term matrix, rows correspond to documents in the collection and columns correspond to terms. We combine it with basic cleaning operations, via functions tm provides us with and a custom stopwords function. The DocumentTermMatrix() function helps us to break up the text into words:

```r
# Stop-words (and, or ... you get the point)
stopwords2 = function (t) {removeWords(t, stopwords())}
# test stopwords function:
stopwords2("He went here, and here, and here, or here") # [1] "He went ,  ,  ,  "


# DocumentTermMatrix
hil_bill_dtm <- DocumentTermMatrix(hil_bill_corpus, control = list(tolower = T, #  all lower case
                       removeNumbers = T, #  remove numbers
                       stopwords2 = T, #  stopwords function
                       removePunctuation = T, #  remove .
                       stemming = T,
                       # Stemming reduces each word to its root from. Imagine you have words in the corpus such as "deported" or "deporting". Appliing the function would result in "deport" only, stripping the suffix (ed, ing, s ...).
                       stripWhitespace = T
                       # removes whitespaces or reduces them to only one each
                       ))
hil_bill_dtm
                       Non-/sparse entries: 94330/1356110
                       Sparsity           : 93%
                       Maximal term length: 45
                       Weighting          : term frequency (tf)
```

Lets split the data up into the training and test data, and save the outputs ("Bill" or "Hillary") as a separate dataframe

```r
hil_bill_dtm_train <- hil_bill_dtm[1:80, ]
hil_bill_dtm_test <- hil_bill_dtm[81:135, ]

# Actual output - whether it was Bill's or Hillary's speech                        
hil_bill_train_labels <- bill_hillary_random[1:80, ]$type
hil_bill_test_labels <- bill_hillary_random[81:135, ]$type
```

Now we have all the data in a clean from to train our model. Lets have some fun, and visualise term frequency via a wordcloud:

```r
library(RColorBrewer)
library(wordcloud)
# filter the data for a word cloud
 hill <- subset(bill_hillary_random, type == "Hillary")
 bill <- subset(bill_hillary_random, type == "Bill")
 wordcloud(bill$text, max.words = 50, scale = c(3, 0.5)) # give us 50 most common words, you can do the same for Hillary's speeches
```

![alt text](/images/naive/plots/bill_wordcloud.png)

Classic Bill, he uses all the disciplinary words I learned to hate. Anyway, lets move on to build the model. For this we can reduce the number of words that are being taken into account as features for the model to words that appear more than a certain amount of times. Here we reducing it to at least 6 times. To do this, you can use the findFreqTerms() function by the tm package. Hereby we reduce the number of features down to only the ones that are making a difference to the probabilities, and down to 1,674 features.

```r
hil_bill_freq_words <- findFreqTerms(hil_bill_dtm_train, 12)
str(hil_bill_freq_words)
# show the structure of 2504
chr [1:1674] "abandon" "abil" "abl" "abov" "abroad" "absolut" "abus" "accept" "access" "accid" "accomplish" ...
```

Now the DocumentTermMatrix's features, the words in the text documents (each speech), needs to be filtered according to the most frequent terms we just worked out via findFreqTerms(). We build a convert funciton to set categorical features to "Hillary" if it is her speech, otherwise it's Bill's. Lastly we convert to a data-frame.

```r

# Filter most frequent terms appearing (for test and training data):
hil_bill_dtm_freq_train <- hil_bill_dtm_train[, hil_bill_freq_words]
hil_bill_dtm_freq_test <- hil_bill_dtm_test[, hil_bill_freq_words]

# the DTM needs to be filled now
convert_counts <- function (x) {
                              x <- if_else(x > 0, "Hillary", "Bill")
                              }
# Apply convert function to get categorial output values
hil_bill_train <- apply(hil_bill_dtm_freq_train, MARGIN = 2,
convert_counts)
hil_bill_test <- apply(hil_bill_dtm_freq_test, MARGIN = 2,
convert_counts)

#convert to df to see whats going on:
hil_bill_train_df <- as.data.frame(hil_bill_train)
View(hil_bill_train_df)
```

![alt text](/images/naive/plots/df.png)

Lets train a model by using the package e1071 to apply the naiveBayes function.

```r
# Train the model:
library(e1071)
library(gmodels)

hil_bill_classifier <- naiveBayes(hil_bill_train, hil_bill_train_labels, laplace = 0) # we set the laplace factor to 0
```

```r
# predict with test data:
hil_bill_test_pred <- predict(hil_bill_classifier, hil_bill_test)

# See how well model performed via cross table:
CrossTable(hil_bill_test_pred, hil_bill_test_labels,
prop.chisq = F, prop.t = F,
dnn = c("Predicted", "Actual"))
```

![alt text](/images/naive/plots/outcome_bill_hill.png)

So here we have it.

The table reveals that in total 1 out of 55 were miss-classified, or 0.018% percent of the speeches (1 that were actually Bills got miss-classified as Hillary's, and 0 of Hillary's got incorrectly classified as Bills). Naive Bayes is the standard for text classification and may have also to a cirtain extent proven that Hillary might be unique to what she says, and doesn't copy from her husband's speeches.

While being First Lady of the United States during the presidency of Bill Clinton from 1993 to 2001, and First Lady of Arkansas during his governorship from 1979 to 1981 and from 1983 to 1992, she now doesnt not simply copy Bill's phrases, but may have developed her own political language.

# Obama vs. Hillary:

![alt text](/images/naive/plots/o_h.jpeg)

Lets try the same spiel with Obama's speeches. We run the model on 107 of Hillary's 2008 presidential campaign speeches and her 96 2016 speeches.

```r
Hillary   Obama
  96       45
```

![alt text](/images/naive/plots/outcome_obama.png) NB performs well, but not as well. With only a 4.87% of miss-classified instances, the model seems to be working reliably.

# Hillary vs. her former self:

While she might have lost the Democratic nomination to Barack Obama in 2008, she became Secretary of State. Leaving office after Obama's first term, she undertook her own speaking engagements before announcing her second presidential run in the 2016 election. So in theory, we should be able to provide evidence whether or not there is a significant difference in her style comparing speeches before her speaking engagement tours and now after, now for her campaign speeches? Lets try and experiment with Hillary's past.

As people know, she successfully served as the 67th United States Secretary of State from 2009 to 2013\. We will take her 2009 speeches and combine it with the ones for her current campaign (but, shorter, since its basically the same thing than above). Here is the link for the csv file.

```r
library(NLP)
library(tm)
library(e1071)
library(dplyr)
library(gmodels)

# read in Clinton_2009 speeches (Info from the US state department), and clean it:
regex <- "^.*:"
Clinton_2009 <- read.csv("download/clinton_2009.csv", stringsAsFactors = F)
View(Clinton_2009_fin)
Clinton_2009_fin <- Clinton_2009 %>%
  filter(str_detect(years, "2009")) %>%
  filter(kind == "Remarks") %>%
  mutate(type = "Clinton_2009")  %>%
  select(-years, -kind, -title) %>%
  mutate(text = str_replace_all(text, "SECRETARY CLINTON", "")) %>%
  mutate(text = str_replace_all(text, "MODERATOR", "")) %>%
  mutate(text = str_replace_all(text, "QUESTION", "")) %>%
  mutate(text = str_replace_all(text, "^[^,]+\\s*", "")) %>% # use speech sample from the first comma onwards
  filter(nchar(text) > 10) # remove empty rows
final_clintons2 <- rbind(clinton, Clinton_2009_fin) # combine the data with Hillary's speeches in the code previously

# Randomize the sample:
set.seed(111) # set seed, to reproduce example
hil_old <- sample_n(final_clintons2, 499)
hil_old<- hil_old %>%
  mutate(type = factor(type))
table(hil_old$type) # we have 403 of clintons 2009 speeches, and almost 100 of her current campaign speeches:
# Clinton_2009      Hillary
#         403           96
hil_old_corpus <- VCorpus(VectorSource(hil_old$text))

# clean and DocumentTermMatrix:
hil_old_dtm <- DocumentTermMatrix(hil_old_corpus, control = list(tolower = T,
                                                                   removeNumbers = T,
                                                                   stopwords = T,
                                                                   removePunctuation = T,
                                                                   stemming = T,
                                                                   stripWhitespace = T
))
hil_old_dtm_train <- hil_old_dtm[1:400, ]
hil_old_dtm_test <- hil_old_dtm[401:499, ]
hil_old_train_labels <- hil_old[1:400, ]$type  
hil_old_test_labels <- hil_old[401:499, ]$type


# Model building:
hil_old_freq_words <- findFreqTerms(hil_old_dtm_train, 5) # features restricted to an appearance of at least 5 times

convert_counds <- function (x) {
  x <- if_else(x > 0, "Hillary_2016", "Hillary_2009")
}
hil_old_dtm_freq_train <- hil_old_dtm_train[, hil_old_freq_words]
hil_old_dtm_freq_test <- hil_old_dtm_test[, hil_old_freq_words]
hil_old_train <- apply(hil_old_dtm_freq_train, MARGIN = 2,
                        convert_counds)

hil_old_test <- apply(hil_old_dtm_freq_test, MARGIN = 2,
                       convert_counds)
hil_old_classifier <- naiveBayes(hil_old_train, hil_old_train_labels, laplace = 0)
str(hil_bill_classifier)
hil_old_test_pred <- predict(hil_old_classifier, hil_old_test)
CrossTable(hil_old_test_pred, hil_old_test_labels,
           prop.chisq = F, prop.t = F,
           dnn = c("Predicted", "Actual"))
```

![alt text](/images/naive/plots/outcome_oldHill_hill.png)

What we see, despite showing somewhat of the same terms (word-clouds below), the model guessed only in about 5% of the speeches wrong, where Clinton's 2009 speeches were miss-classified as her 2016 ones.

```r
hill <- subset(hil_old, type == "Hillary")
Old_Hillary <- subset(hil_old, type == "Old_Hillary")
library(RColorBrewer)
library(wordcloud)
wordcloud(hill$text, max.words = 30, scal = c(3, 0.2))
wordcloud(Old_Hillary$text, max.words = 30, scale = c(3, 0.5))
```

## Classifying Hillary's presidential speeches (2008 vs. 2016)

![alt text](/images/naive/plots/oldHill_wordcloud.png)

The speeches as Secretary of State might not be perfect to compare her style for the 2016 presidential el estion. One should mind to compare apples with oranges. To find a possibly more closely related dataset, Hillary's presidential campaign speeches from her 2008 presidential election campaign might serve us well. Again, fetched data from the web, this time from the [UCSB page](http://www.presidency.ucsb.edu/2008_election_speeches.php?candidate=70&campaign=2008CLINTON&doctype=5000) . We see our model in action on the following instances:

```r
Clinton_2008_Presidential_Election                       Hillary_2016
107                                 96
```

![alt text](/images/naive/plots/outcome_oldHill_presidential_08_hill.png) Remarkable! Our NB model, with an accuracy of 98%, performed really well and was able to spot the differences between Clinton's 2016 campaign and her campaign speeches she gave in 2007 and 2008.

# Running a model aginst Hillary's tweets:

In order to run a NB model against her tweets, we need text that Hillary didn't wrtie. The blog may be a good start. I have written a scraper to get the last 100 blog entries. The data will be trained on 89 blog entries (cleaned up, so we don't include Spanish entries here), and Hillary's speeches of he 2016 campaign again. When we sort out the messages that Hillary signs with an "-H" or "-Hillary", then we know they are hers. With this in mind, we build a new model (as above). We take her blog posts, and her speeches, and train a NB model. Next, buld a test set with her tweets, and test labels, where Hillary signed with her name.

![alt text](/images/naive/plots/output_tweets.png)

We run our classifier on it the tweets and notice that our model got it completely wrong. What a shame. This could mean many things, including that the test data wasn't proberly cleaned. 47% were wrongly classified where it thought that Hillary written the tweet, while for 8 tweets by Hillary, the classifier assumed it was Clinton's team.

![alt text](/images/naive/giphy.gif)

# Conclusion:

Here is an overview how well the model performed on various speakers and texts:

![alt text](/images/naive/plots/conclusion.jpg) Overall, Naive Bayes for text - in our case speech - classification is a powerful tool. The biggest hurdle is to keep the data clean, while the another challenge is to find a useful use-case. If a model is being run, then on somewhat comparable data text sources (e.g. same time, or text someone that is in relation with someone else). For Hillary Clinton, the model showed that she is her own persona, her speeches reflect her, not her husband. Her campaign is her unique voice. When it comes to tweets, the NB model had problems to separate them correctly. This might not be a terrible thing for her campaign. The more her campaign team's tweets and appearances align with her own speeches, the more unite the campaign effort.

[5b26ae86]: https://www.hillaryclinton.com/ "link"
