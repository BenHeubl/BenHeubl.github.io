---
title: This is why guns are an inherent part of the movie industry
header:
  teaser: 'https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg'
categories:
  - Data analysis
tags:
  - update
---

**Guns are as much part of the american film indsutry, as cool cars or good looking people. But a closer look at scraped date reveals important insights on the close relationship between a gun manufactioring indstry and their lobby, their company brandings and the collabroation with the film indsutry.**

![alt text](/images/guns/header.png)

Coming across this Internet Movie Firearms Database, I was eager to explore the data. To fetch data for a basic overview of actors and gun appearances, I wrote the following script in R:

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

# Calculate how many times the loop has to run:
ntimes <- round(11182/200)
# get all the 200_link links:
df_200 <- NULL
ink <- "http://www.imfdb.org/index.php?title=Category:Actor&pagefrom=A.+Russell+Andrews#mw-pages"
for (sub in 1:ntimes) {
  df_200 <- rbind(df_200, as.data.frame(ink))
  ink <- paste("http://www.imfdb.org/", get_200_link(ink), sep = "")
  print(ink)
  }

########## 2\. Function to scrape links from "next200" links on each page:
get_200_each <- function (url) {
  get_sub <- read_html(url)
  link_200_sub <- get_sub %>%
    html_nodes("#mw-pages .mw-content-ltr a") %>%
    html_attr("href")
  return(link_200_sub)
}
# test: get_200_each("http://www.imfdb.org//index.php?title=Category:Actor&pagefrom=Xavier+Hosten#mw-pages")
sub_sub_200 <- NULL
for (sub_sub in 1:nrow(df_200)) {
  url_sub <- as.character(df_200[sub_sub, 1])
  print(url_sub)
  sub_sub_200 <- rbind(sub_sub_200, as.data.frame(get_200_each(url_sub)))
}

######## 3\. get all the individual information on each actors page:
get_info <- function (actor_url) {
  ac <- read_html(actor_url)
  gun <- ac %>%
    html_nodes('#mw-content-text td:nth-child(1)') %>%
    html_text()
  gun_link <- ac %>%
    html_nodes('#mw-content-text td:nth-child(1) a') %>%
    html_attr("href")
  gun_final <- paste("http://www.imfdb.org" , gun_link, sep = "")
  character <- ac %>%
    html_nodes('#mw-content-text td:nth-child(2)') %>%
    html_text()
  movie <- ac %>%
    html_nodes('#mw-content-text td:nth-child(3)') %>%
    html_text()
  movie_link <- ac %>%
    html_nodes('#mw-content-text td:nth-child(3) a') %>%
    html_attr("href")
  movie_final <- paste("http://www.imfdb.org" , movie_link, sep = "")
  note <- ac %>%
    html_nodes('#mw-content-text td:nth-child(4)') %>%
    html_text()
  ###
  date <- ac %>%
    html_nodes('#mw-content-text td:nth-child(5)') %>%
    html_text()
  name<- ac %>%
    html_node("#firstHeading span") %>%
    html_text()
  all_return <- cbind(data_frame(name), data_frame(gun), data_frame(gun_final), data_frame(character), data_frame(note), data_frame(movie), data_frame(movie_final), data_frame(date))
  return(all_return)
}
#(get_info("http://www.imfdb.org/wiki/Al_Shannon"))

require(plyr)
final_data_guns <- NULL
for (t in 2:nrow(sub_sub_200)) {
  tryCatch({
    link <- paste("http://www.imfdb.org", sub_sub_200[t, 1], sep = "")
        print(link)
        final_data_guns <- rbind.fill(final_data_guns, as.data.frame(get_info(link)))
  }, error=function(e){cat("ERROR :",conditionMessage(e), "\n")})
}
```

Above we created the crawling functions, and several looping functions that apply them to scrape data from each sub-page. To get access to all the links for each movie, we used the "next200" link buttons on the page, which always contains another link to more sublinks.

Next, we are interested in each movie, its directors, distributors and release years. As we have now access to all these links, we can once more go through all the links and scrape the information from each movie page, and later use inner join to connect the data via the line id we arrange for.

```r
g_new <- g %>%
  mutate(id = rownames(g))  # creates a simple id
library(rvest)
library(dplyr)
library(tidyr)
library(xml2)
get_info <- function (url) {
  r <- read_html(url)
  des0 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(1) th:nth-child(1)") %>%
    html_text()
  data0 <- r %>%
    html_nodes("td tr:nth-child(1) th+ th") %>%
    html_text()
  zero <- (paste0(des0, data0, sep = "_"))
  des1 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(5) th:nth-child(1)") %>%
    html_text()
  data1 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(5) th+ th") %>%
    html_text()
  one <- (paste0(des1, data1, sep = "_"))
  des2 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(2) th:nth-child(1)") %>%
    html_text()
  data2 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(2) th+ th") %>%
    html_text()
  two <- (paste0(des2, data2, sep = "_"))
  des3 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(3) th:nth-child(1)") %>%
    html_text()
  data3 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(3) th+ th") %>%
    html_text()
  three <- (paste0(des3, data3, sep = "_"))
  des4 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(4) th:nth-child(1)") %>%
    html_text()
  data4 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(4) th+ th") %>%
    html_text()
  four <- (paste0(des4, data4, sep = "_"))
  des5 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(6) th:nth-child(1)") %>%
    html_text()
  data5 <- r %>%
    html_nodes("td td table:nth-child(1) tr:nth-child(6) th+ th") %>%
    html_text()
  five <- (paste0(des5, data5, sep = "_"))
  info <- cbind((zero), (one), (two), (three), (four), (five))
  return(info)
}
df_guns2 <- NULL
### scrape data for movies specifically
for (m in 1:nrow(g_new)) {
  tryCatch({
    linkss <- as.data.frame(get_info(g_new[m, 8]))
    id <- g_new[m, 10]
    #bounds <- cbind(bounds, linkss, as.data.frame(ids))
    df_guns2 <- rbind(df_guns2, cbind(linkss, as.data.frame(id)))
    print(m)
  }, error=function(e){cat("ERROR :",conditionMessage(e), "\n")})
}
```

As the features in the tables on each movie page may not always comes in the same order, we can push all features into a single column, and extract it via some simple regex. Next we join our previous dataset. Voila! We have our final dir_join dataframe, and can start with the analysis.

```r
dir_all <- dir %>%
  distinct(id, .keep_all = TRUE) %>%  # get distinct lines from id column, but keeps all of them.
# remove duplicate rows  
  mutate(all = paste0(V1, V2, V3, V4, V5, V6, sep = "_")) %>%  # collects all in one row, as character string
  mutate(studio = str_extract(all, "Studio\\W([a-zA-Z W])+")) %>%
  mutate(studio = str_replace(studio, "Studio", "")) %>% # Which studio was it filmed in?
  mutate(directed = str_extract(all, "Directed by\\W([a-zA-Z. W])+")) %>% # Which director?
  mutate(directed = str_replace(directed, "Directed by", "")) %>%
  mutate(Release = str_extract(all, "Release Date\\W([a-zA-Z.\\d W])+")) %>% # Release Date
  mutate(Release = str_replace(Release, "Release Date", "")) %>%
  mutate(distributor = str_extract(all, "Distributor\\W([a-zA-Z.\\d W])+")) %>% # Distributor
  mutate(distributor = str_replace(distributor, "Distributor", ""))  
dir_join <- g_new %>%
  inner_join(dir_all)
```

# Could more different guns in movies mean a response to a more gun loving audience?

First off, I am interested in how many movies per year with gun appearances are listed in the dataset. I also want to know the average variety of guns for each year. Guns with a high average variety may indicate that the industry paid more attention to detail when it comes to the different type. Could mean a response to a more gun loving audience? A hyperthesis that should be further investigated.

![pic1]({{ site.url }}/images/guns/plots/years_number.svg)

We can see that movies produced in the periods of 1967 and 1969 feature many different gun types, and a spike in gun featuring movies. The industry produced a range of anti war movies such as "How I Won the War" from 1967\. The spike in the 60ties comes at no surprise. On the contrary, the spike between 2010 and 2014 is truly surprising, and an average of gun diversity in 2015 of 5.32 has not bees as high since 1970\. Is the gun featuring movie having a revival?

# Whose fault is that: Producers and Studios?

Who put the guns into movies? Script writers, producers and the film production as a whole assume are responsible for the diecisions related to what guns appear in the movie. Lets have a look at studios, that produced a big junk of gun featuring movies: Warner Bros. is one example. The date features 50 movies by Warner.

![pic1]({{ site.url }}/images/guns/plots/Warnerfilms.svg)

While there are a few outliers, Warner bro. doesn't seem to follow a specific strategy. However, plotted against their biggest rivals, the studio doesnt seem to feature a greater variety of guns in their film over the past decades. Other major studios however seem to have changed their strategies however.

![pic1]({{ site.url }}/images/guns/plots/warner_competition.svg)

While Disney seems to have increasingly featuring a larger variety of gun types on their movies, other studios seem to do so less so.

# How does this connect to ratings?

While it is interesting to look at this data on its own, it is worthwhile to consider what the movies industry really cares about. One important measure are ratings. IMDB is one of the first sources i checked out. I scraped the data fro action movies back to 2011\. Here how movie ratings (not) correlate with the number of gun types appearing in each movie:

# ...?

... ![pic1]({{ site.url }}/images/guns/plots/studio_analysis.svg)

# ...?

... ![pic1]({{ site.url }}/images/guns/plots/movies_years_gun_appearances.svg)

# ...?

... ![pic1]({{ site.url }}/images/guns/plots/producers_years_gun_appearances_frequent.svg)
