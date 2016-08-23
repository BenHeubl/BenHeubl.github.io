---
title: Fashion models
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - Jekyll
tags:
  - update
---

# Models are hardly representing the general public

**The Fashion Model Directory is an online database of information about fashion models, modelling agencies, fashion labels, fashion magazines, fashion designers, and fashion editorials. FMD has been described as "the IMDb of the fashion industry" as one of the largest online fashion databases. Started as an offline project in 1998 by Stuart Howard, FMD went live on the web in 2000 and was taken over by British media group Fashion One Group two years later**.

Discovering this database, I was eager to explore it. So i went and written a small scraper, with the help of Hadley Wickham's beautiful RVEST.

```r
require(rest) #rvest-IloveYou
get_link <- function (url_ABC) {
  got_url <- read_html(url_ABC)
  models_link <- got_url %>%
    html_nodes(".Portrait a") %>%
    html_attr("href")
  return(models_link)
}
get_individual <- function (sub_url) {
  ind <- read_html(sub_url)
  models_each <- ind %>%
    html_nodes(".Portrait a") %>%
    html_attr("href")
   return(as.character(models_each))
}
get_it <- function (sub_url, ids) {
  ind_it <- read_html(sub_url)
  height_it <- ind_it %>%
    html_node(".Height .Separator+ .DataPoint") %>%
    html_text()
  hair_m <- ind_it %>%
    html_node(".Hair .Wrap") %>%
    html_text()
  eye_m <- ind_it %>%
    html_node(".Eyes .Wrap") %>%
    html_text()
… # other features
  content_1 <- as.data.frame(height_it, as.character(ids))
  content_2 <- cbind(content_1, eye_m, bust_m)
  content_3 <- cbind(content_2, waist_m, hips_m, hair_m)
  content_4 <- cbind(content_3, names, nat, birth, sub_url)
  return(content_4)
}
```

After setting up the scraper functions, the script should run through all the pages individually. I am sure there is a more effective way of doing this (python's beautiful soup is a faster alternative), but it also works in R.

```r
Each_model_link <- NULL
for (b in letters) {
  link <- paste("http://www.fashionmodeldirectory.com/models/search/alphabetical_order/", toupper(b), sep = "")
  Each_model_link <- rbind(Each_model_link, as.data.frame(link))
}
number_subs <- NULL
for (sub in 1:nrow(Each_model_link)) {
  urlABCs <- as.character(Each_model_link[sub, 1])
  readABC <- read_html(urlABCs) %>%
    html_node(".CroppingDescription:nth-child(1) .Color_Page+ .Color_Page") %>%
    html_text()
  number_subs <- rbind(number_subs, cbind(as.data.frame(readABC), urlABCs))
}
p <- number_subs %>%
  mutate(pages = as.character(readABC)) %>%
  filter(!is.na(pages)) %>%
  select(pages, urlABCs)
pages_all <- NULL
for (l in 2:nrow(p)) {
  as.data.frame(dates), as.data.frame(to))))
  multy = round(as.numeric(p[l, 1])/12)
  pages_letter = seq(1, multy)
  for (t in 1:length(pages_letter)) {
    insert = t * 12
    urli <- paste0((p[l, 2]), "/?start=" , insert, sep = "")
    pages_all <- rbind(pages_all, as.data.frame(urli))
    }
}
all_urls2 <- NULL
for (h in 2:nrow(pages_all)) {
   linkU = pages_all[h, 1]
   all_urls2 <- rbind(all_urls2, as.data.frame(get_individual(as.character(linkU))))
   print(h) # counter
}
df_final <- NULL
for (e in 2:nrow(all_urls2)) {
  fin = all_urls2[e, 1]
  df_final <- rbind(df_final, as.data.frame(get_it(as.character(fin), e)))
}
```

If your scrip made it all the way through, you should have a beautiful dataset now available as df_final. Lets explore it.

## Fashion models are far off of being representative:

Interesting is the relationship between height and waist. First we will clean it, then plot it and compare it with the average waist sizes and heights for females, and requirements model experts claim.

```r
f_clean_0 <- f %>%
  separate(bust_m, c("bust_1", "bust_2"), sep = "/", na.rm = T) %>%
  separate(waist_m, c("waist_1", "waist_2"), sep = "/", na.rm = T) %>%
  separate(hips_m, c("hips_1", "hips_2"), sep = "/", na.rm = T) %>%
  separate(height_it, c("height", "del"), sep = "cm", na.rm = T) %>%
  select(-del) %>%
  mutate(cm = as.numeric(height)) %>%
  filter(!is.na(height), !is.na(waist_2)) %>%
  mutate(waist_cm = as.numeric(waist_2)) %>%
  mutate(w_h = waist_cm/cm) %>%
  mutate(hips_cm = as.numeric(hips_2)) %>%
  arrange(desc(cm)) %>%
  filter(waist_cm < 200, waist_cm > 40) %>%
  mutate(born_in = str_extract(birth, "(19|20)\\d\\d")) %>%
  mutate(ages = 2016-as.numeric(born_in)) %>%
  mutate(over30 = ifelse(ages>30, "<30", ">30"))  
ggplot(f_clean_0, aes(cm, waist_cm)) +
  geom_jitter(show.legend = F, size = 0.5, aes(col = ifelse(underweight == 1, "red", "navyblue"))) +
  ggtitle("Waist-Heigth distributions of models on Fashionmodeldirectory.com") +
  ylab("Waist in cm") +
  xlab("height in cm") +  
  theme_bw() +
  scale_colour_manual(values = c("blue","red", "green")) +
  geom_vline(xintercept = 163, col = "blue") +  # UK female hight
  geom_vline(xintercept = 158.9, col = "red") +  # US
  geom_hline(yintercept = 83, col = "blue") + # UK female waist
  geom_hline(yintercept = 99, col = "red") # US female waist
```

Health professionals say that for women, being severely underweight can result in amenorrhea (absence of menstruation), infertility and possible complications during pregnancy. It can also cause anemia and hair loss, while being underweight is an established risk factor for developing [osteoporosis](https://en.wikipedia.org/wiki/Osteoporosis), even for young people. One typical measure is the BMI (the underweight definition usually refers to people with a body mass index (BMI) of under 18.5 or a weight 15% to 20% below that normal for their age and height group).

A person may be underweight due to genetics, metabolism, drug use, lack of food, or illness. Can being a fashion model represent another factor for being unhealthy underweight too? We can't really tell with the BMI, as the data doesn't contain the measure of weight in the model's profiles. However, another measure can help us. The so called waist-height ratio: A measurement of the waist circumference divided by height (both in cm). [Online sources](waisttoheightratio) argue that a female waist between 0.35 - 0.42 would count as "extremely slim", while being below may serve us here for a guidance of being underweight (whether that is clinically absolutely correct, would need to be established).

![pic1]({{ site.url }}/images/models/plots/A.svg)

We clearly see how the "underweight" group of models show up in the data (in red circles). The number of models that accounted for a waist-height ratio of below 0.35 was stunning: 73.9%, or 7.221 women out of 9.732 fashion models complied with the categorisation (to be fair only about 41 girls fell under the 0.30 w/h mark). To compare to the average women in the age groups our models fit, the lines show population averages for the US (in red) and the UK (in blue).

> Experts suggest that waist-to-height ratio may be a better screening tool than waist circumference and BMI for adult cardiometabolic risk factors (according to Ashwell, M., Gunn, P. & Gibson, S., 2012). How good this measure is as an indicator for unhealthy underweight and eating disorders, couldn't be established.

One of the first things I usually look at, at this stage, are correlations among variables. One neat helper function comes with the psych package:

```r
library(psych)
pairs.panels(f_clean_0[c("waist_cm", "cm", "hips_cm", "ages")])
```

![pic2]({{ site.url }}/images/models/plots/C.svg)

We can see that we have a low correlation value between height (cm), and their waist. This hint to the fact event tall models have to compete with the same waist sizes than less tall models. The correlation between hips and waist is fairly logical. If we wanted to, we could now establish a classification model to train our data on. However for this exercise, we won't (For a future blog post, I will look into it myself, but for now, feel more than free to use and play around with the data yourself).

## TO HAVE WHAT IT TAKES IS HARD, EVENT FOR MODELS

Modelling requirements and the rules of the game are tough, as [this](http://modelingwisdom.com/height-age-and-measurement-requirements-of-modeling) fashion model blog tells us. To have what it takes, women need to be of the "typical age is 16-21", have a bust between 32″-36″, a waist between 22″-26″, and a Height between 5'9″-6″ (175 - 182 cm). Let's test these made up rules on our data. First on the waist and height requirements:

![pic3]({{ site.url }}/images/models/plots/requA.svg)

In terms of waist and height, only 6.528 women out of 9.696 models would comply with the "rules" above. Does this mean that if these 32.6 % of models weren't already successful professionals and they would now apply to agencies, they wouldn't be accepted? If you ask me, these requirements smell fishy. Let's test them on the bust and hip-size requirements.

![pic4]({{ site.url }}/images/models/plots/requB.svg)

Here again a similar picture: Only a bit more than one-half of all women, about 5,978 out of 9,696, with a bust size between 32″-36″ and hips size between 33″-35″ could meet [their](http://modelingwisdom.com/height-age-and-measurement-requirements-of-modeling) requirements.

### Ages and origins:

Next, it would be interesting how fashion models score in regards to their nationality and their ages. For that, we need to group them by their origin.

![pic5]({{ site.url }}/images/models/plots/D.svg)

We learn that while Spanish, Italian and French models stick out with a median age of over 30, Dutch, Russian, Canadian, Australian, Brazilian models fall below this threshold (only a fraction of model profiles had age actually as a valid data point).

We can also look at both, the measure of weight and hight and nationalities.

```r
f_waist <- f_clean %>%
  select(nat, waist_cm, cm, hips_cm, ages, underweight, w_h)
f_waist_sum <- f_waist %>%
  group_by(nat) %>%
  summarise(waist = median(as.numeric(waist_cm)), height = median(as.numeric(cm)), N = n(), uw = sum(underweight)/N, uw_mean = mean(w_h)) %>%
  arrange(desc(waist))
ggplot(f_waist_sum, aes(waist, height, label = nat, col = uw_mean)) +
   geom_jitter(aes(size = N)) +
   geom_text(show.legend = F, check_overlap = T) +
   ggtitle("Waist vs. height for different nationalities") +
   theme_bw() +
   xlab("Waist size in cm") +
   ylab("Height in cm") +
   scale_colour_gradient(limits=c(0.340, 0.37), low="red", high = "grey") +
   ggsave("/Users/BH/BLOG/BLOG_DATA/source/images/posts/models/plots/E.svg", width = 20, height = 20, units = "cm")
```

![pic5]({{ site.url }}/images/models/plots/E.svg)

The more red, the lower is the average waist-height ratio and the more alarming the statistics for the countries with a representative sample size.

### The End:

To conclude, we learned how to scrape data with RVEST (you can also download the data here), how to understand what sort of BS model blog can claim in regards to industry's model requirements and what the real representation among professional models for these made up rules are.

Being well aware that this is only a start of an investigation, future data exploration could look at how a representative sample of women (in terms of age) from the general public compares to these models, how origin specific average measurements compare to the model data, and - most interesting in my view - how models measurements changed over the course of the last ten to twenty years. This [source](http://abcnews.go.com/blogs/headlines/2012/01/most-models-meet-criteria-for-anorexia-size-6-is-plus-size-magazine/) claims that twenty years ago, the average fashion model weighed 8% less than the average woman while today she would weighs 23% less (rader Programs, which treats those with eating disorders).

It would be worthwhile to be pointed in the right direction if you know where to get fashion model micro data from the past (reach out on twitter (@benheubl)).

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
