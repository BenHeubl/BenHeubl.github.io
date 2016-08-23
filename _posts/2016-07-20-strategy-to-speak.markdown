---
title: Fashion models
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - Jekyll
tags:
  - update
---

**Text analysis in R reveals how Hillary Clinton structured her conventional speeches, how Clinton's remarks at the US State department from 2009 to 2013 reveals important hints on how she could govern the country if she is being elected into the White House, and what Clinton's tweets reveals about her campaign strategy**.

Great, so the convention is over. Clinton will go against Trump in the election finals. Now what? After Trump's and Clinton's speeches at the conventions, people are left with impressions. Listeners to conventional speeches noted how negative Trump's speech was. Clinton was generally said to be balanced, except at times when she discussed how truly unfit her opponent is for presidency.

To understand Hillary's sentiment in her speeches, we use [Julia Silge's and David Robinson's tidytext](https://www.r-bloggers.com/the-life-changing-magic-of-tidying-text/). First we load in the tidytext package, dplyr and stringr for some basic data wrangling. For the analysis of conventional speeches by democrate candidates, we set a linenumber bring the data into the format we need it in.

```r
library(tidytext)
library(dplyr)
library(stringr)
Speeches.19_clean_Democrats <- Speeches.19_clean %>%
   filter(party == "Democratic")
Speeches.19_clean_Republican <- Speeches.19_clean %>%
  filter(party == "Republican")
conv.all_Dem <- Speeches.19_clean_Democrats %>%
  group_by(title) %>%
  mutate(linenumber = row_number()) %>%
  ungroup() %>%
  separate(title, c("speaker", "Years"), sep = "_", remove=FALSE)
```

The next thing is to "untaken" the text into words that we can analyse it. We also load in stop words and use dplyr's "anti-join" to clean the data. Another thing we arrange is to involve the bing lexicon dataset, which will help us with the clarification of a word's sentiment.

```r
tidy_All_Dem <- conv.all_Dem %>%
  unnest_tokens(word, text)
data("stop_words")
tidy_All_Dem <- tidy_All_Dem %>%
  anti_join(stop_words)
tidy_All_Dem %>%
  count(word, sort = TRUE)
library(tidyr)
bing <- sentiments %>%
  filter(lexicon == "bing") %>%
  select(-score)
All_sentiment_Dem <- tidy_All_Dem %>%
  inner_join(bing) %>%
  count(title, index = linenumber %/% 1, sentiment) %>%
  spread(sentiment, n, fill = 0) %>%
  mutate(sentiment = positive - negative) %>%
  separate(title, c("speaker", "Years"), sep = "_", remove=FALSE)
```

Now we are good to go to plot (full code can be found in the data repo).

```r
library(ggplot2)
library(viridis)
library(grid)
library(directlabels)
All_sentiment_standard_Dem$title <- factor(All_sentiment_standard_Dem$title,
levels = (All_sentiment_standard_Dem$title[order(All_sentiment_standard_Dem$Years)]))
ggplot(All_sentiment_standard_Dem, aes(index_stan, sentiment, group = title)) +
  scale_colour_brewer(palette = "Set1") +
  geom_path(show.legend = F, alpha = 0.3, linejoin = "mitre", lineend = "butt", aes(col = if_else(hillaryClinton_line == 0, "red", "grey"))) +
  facet_wrap(Years~speaker, nrow = 2, scales = "free_x") +
  theme_minimal(base_size = 13) +
  labs(title = "Sentiment in democratic US presidential candidates convention speeches",
       y = "Sentiment") +
  geom_smooth(method = "loess", n = 50, show.legend = F, aes(col = if_else(hillaryClinton_line == 0, "red", "grey"))) +
  scale_fill_viridis(end = 0.75, discrete=TRUE, direction = -1) +
  scale_x_discrete(expand=c(0.02,0)) +
  theme(strip.text=element_text(hjust=0)) +
  theme(strip.text = element_text(face = "italic")) +
  theme(axis.title.x=element_blank()) +
  theme(axis.ticks.x=element_blank()) +
  theme(axis.text.x=element_blank()) +
  geom_dl(aes(label = toupper(Years)), method = list(dl.trans(x = x - 1.3), "last.points", cex = 0.5))
```

![pic1]({{ site.url }}/images/strategy-to-speak/plots/conventionsHillary.svg)

> The plot shows the sentiment analysis for democrate conventional speeches over the past years. A blue weighted loess line is introduced that shows a general trend over the course of each speech. Clearly Hillary's speech is rather balanced. We can do the same thing for the republican conventional speeches and witness one of the most negative talks over the past decades by candidate Trump.

![pic1]({{ site.url }}/images/strategy-to-speak/plots/conventionsTrump.svg)

> Length: Over the past three decades, conventional speeches significantly varied in their length and while Bill Clinton's conventional speech listeners in 1996 must have a hard time to stand his 7,000 words, other candidates such as Mondale kept it brief down to 2400.

# Hillary in office, what's it like?

I am interested in Hillary's past. One way to investigate is to check what she said when she filled the job as US Secretary of State at the state department from 2009 to 2013\. First lets get all the remarks she gave loaded into the console:

```r
remakrs <- read.csv("link", header = T)
```

# Hillary and Twitter...

wrapup/conclusion

call to action:

# Who is Hillary Clinton?

Next, it was interesting to analyse Hillary's past speech records. It was easy to find her remarks from back when she served as US Secretary of State from 2009 to 2013.

![pic1]({{ site.url }}/images/strategy-to-speak/plots/plot.svg)

> Looking only at Hillary's remarks at the US state department from 2009 to 2013, she gave a lot more speeches in her first year as Secretary of State. While the word international appeared in titles of remarks which where was rated as more positive in 2009, she later must have talked about more serious subjects as the sentiment scores decline.

> As we also know Clinton is and was always a speaker for women's rights. It is not surprising that the word "Women" was part of many speeches she gave as Secretary of State.

hc, as the junior US Senator representing New York from 2001 to 2009, as First Lady Bill's presidency (1993 to 2001), and First Lady of Arkansas during her husband's governorship from 1979 to 1981/1983 to 1992\. So loads of data of speeches.

Turns out there is an R package to measure text sentument analysis as easy as stealing candy from a baby. What of the key questions that has tickled me to research this story was how does today's Hillary Clinton compares to her former self from years ago. To learn more about it, I first concentrated on her time at the US Department of State, from 2009 to 2013\. In the title of her speeches back then, the word "women" appeared many times. Clinton is know to fight for women's rights across the years:

She was also giving many speeches that contained the word "international", however the sentiment of her international speeches may have changed and became more series over the years. This could be due to the fact that Hillary became more confident in talking openly about problems with other countries.

Lastly, speeches that contained "with president" in the title were not given as often i i thought.

## Conventional speeches

If comparing the past years's conventional speeches, showing trump as a neagive outlier:

While the sentiment analysis on Hillary's speech showed evidence of an equilibrium in style for positive and negative sentiment, Trump's seemed to have presented one of the most negative talks across three decades of conventional speeches.

Although Trump started and ended on a high note, he quickly became negative after his initial "we will lead our country back to safety, prosperity, and peace". Negative scores started to appear when Trump discussed that this convention would occur at a moment of crisis, and how attacks on the police and how the terrorism in our cities would threaten people's lives.

Hillary, on the contrary was generally balanced. Her talk got a positive sentiment score when she explained how America would need everyone to lend their energy, talents, ambition to making the nation better and stronger. However, her sentiment score decreases significantly when she questioned Trump's temperament to be commander in chief. "Donald Trump can't even handle the rough and tumble of a presidential campaign", Clinton said.

conclusion? Overall, conventional speakers across the past conventions kept their speech rather positive than negative. To point out, that the status quo is bad, doesn't make a president. Instead, it should matter how good the solutions are the candidate proposes to fix problems. Trump's strategy was clearly to stand out, which he succeeded in. Clinton's talk on the other hand was to discuss facts, which may have made her speech's sentiment more dry and to less fluctuate across its length.

The graph intents to give an idea of the sentiment of Hillary Clinton speeches and her remarks for the time period when she was acted as secretary of state. The small dots represent speeches, the y values the sentiment score the remark was judged on across each corpus of speech text that was published by the US. department of state website. We can see that Hillary Clinton made many more speeches in the beginning of her secretary of state carreer in 2009 than later on. Her sentiment changes over the period of the time. Espicially around key topics such as ... when we encounter a spike in her sentiment.

For a short while i had the chance to be a speaker. I spoke on conferences and meetings, mainly as a journalist. Political leaders of the world use their speech as their core medium to deliver their point of view, their convinctions, and surely to justify topics and actions. The yield is an engaging audience.

Needless to say, that text one of the most exciting data sources, the so to speak upcoming new cool new kid on the blog among data sources, analysing speeches by politiciancs might be an obvious first choice, but can reveal a lot about their style and their intentions.

If you do or did any public speaking, you might find it interesting if and how your style changed over the year.

## Text analysis in R:

<!-- <link rel="stylesheet" type="text/css" href="/javascripts/posts/test/style.css"> <script src="/javascripts/libs/d3.4.11.js" type="text/javascript"></script> <script src="/javascripts/libs/lodash.js" type="text/javascript"></script> <script src="/javascripts/libs/d3-jetpack-v1.js" type="text/javascript"></script> <script src="/javascripts/libs/d3-starterkit-v0.js" type="text/javascript"></script> <script src="/javascripts/posts/test/graphtest.js"></script> -->

<link href="/bootstrap.min.css" rel="stylesheet">



<link href="/clean-blog.css" rel="stylesheet">



<link href="font-awesome.min.css" rel="stylesheet" type="text/css">



<link href="http://fonts.googleapis.com/css?family=Lora:400,700,400italic,700italic" rel="stylesheet" type="text/css">



<link href="http://fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,600italic,700italic,800italic,400,300,600,700,800" rel="stylesheet" type="text/css">



<script src="/jquery.min.js">
</script>



<script src="/bootstrap.min.js">
</script>



<script src="/jqBootstrapValidation.js">
</script>



<script src="/contact_me.js">
</script>



<script src="/clean-blog.min.js">
</script>
