Fun with R
================
Wouter van Atteveldt
September 2021

-   [Installing R & Rstudio](#installing-r--rstudio)
    -   [Installing packages](#installing-packages)
-   [Scraping twitter](#scraping-twitter)
    -   [Some simple graphs](#some-simple-graphs)
-   [Analysing text with Quanteda](#analysing-text-with-quanteda)
    -   [Improving our word cloud](#improving-our-word-cloud)
    -   [Tag cloud](#tag-cloud)
-   [Corpus comparison](#corpus-comparison)

The goal of this handout is simply to have some fun with R. It’s OK if
you don’t understand all commands – just play around, see if you can get
it working, and see if you can change the code to change what it does.
And remember: you can’t break anything!

For more information, see chapter 2 of [Computational Analysis of
Communication](https://cssbook.net).

# Installing R & Rstudio

If you have already installed R and RStudio, you can skip right ahead to
the next section.

To install R, go to the right page on CRAN
([Windows](https://cran.r-project.org/bin/windows/base/),
[Mac](https://cran.r-project.org/bin/macosx/)) and follow the
instructions there. Next, go to
[RStudio](https://www.rstudio.com/products/rstudio/download/#download)
and install it.

After this, you can open RStudio and write your script in the top left
(after selecting File, New File, R Script). By placing your cursor on a
line and pressing control + Enter, that line is executed in the bottom
left and you should see the results.

Please try this out with the following line:

``` r
print("Hello, world!")
```

## Installing packages

Most R Functionality is contained in *packages*. For installing these,
you can use the packages pane in RStudio (bottom right), or simply use
the code below which installs the packages used in this handout:

```
install.packages("tidyverse")
install.packages("quanteda")
install.packages("quanteda.textplots")
install.packages("quanteda.textstats")
install.packages("RColorBrewer")
install.packages("rtweet")
```

Note that you only need to do this once on your computer, so next time
you can skip this step (and if you are submitting code for e.g. an
assignment, there is no need to include the installation calls)

# Scraping twitter

To scrape twitter, we use the `search_tweets` command from the rtweet
library:

``` r
library(rtweet)
tweets = search_tweets("texas", n = 1000, include_rts = FALSE)
```

Note that the first time this will open a browser window in which you
need to login to your twitter account and authenticate it.

This command collects the first 1000 tweets mentioning ‘texas’. Of
course, you can change the search term or use different search options –
see the
[documentation](https://developer.twitter.com/en/docs/twitter-api/v1/rules-and-filtering/search-operators)
for more information on how to search for e.g. usernames, mentions, etc.

After running the command above, there should be a new object “tweets”
in your Environment panel, consisting of 1000 rows In fact, the
`search_tweet` commands includes a *lot* of information on each tweet
(click on the ‘tweets’ object in the Environment pane to inspect it). To
make it a little bit more manageable, we use the *tidyverse* select
command to only keep a subset of columns:

``` r
library(tidyverse)
tweets = select(tweets, status_id, created_at, screen_name, lang, location, retweet_count, text)
tweets
```

If for some reason this does not work, you can download a cached set of
tweets on COVID using the command below:

``` r
tweets = read_csv("https://cssbook.net/d/covid.csv")
```

Note that, of course, twitter research (‘twitterology’) has various
drawbacks: it is only representative of what twitter users tweet about,
it is unclear how the data is curated, etc. However, given the public
nature of the data it is still a very interesting source of information.
If you want to do more serious research with twitter data, consider
using the Academic API which gives fuller access to the data (see the
[academictwitteR](https://github.com/cjbarrie/academictwitteR) package)

## Some simple graphs

In which language are the tweets?

``` r
ggplot(tweets) + geom_bar(aes(x=lang))
```

What date and time are they posted?

``` r
ggplot(tweets) + geom_density(aes(x=created_at))
```

# Analysing text with Quanteda

What would a social media analysis be without a word cloud? The code
below uses the `quanteda` package to create a corpus, tokenize it, and
create a document-feature matrix (`dfm_tweets`). Finally, it uses the
`quanteda.textplots` package to create the wordcloud:

``` r
library(quanteda)
library(quanteda.textplots)
dfm_tweets = tweets %>% 
  corpus() %>% 
  tokens() %>% 
  dfm() 
textplot_wordcloud(dfm_tweets, max_words=100)
```

## Improving our word cloud

Now, this was very simple, but also not very informative: we mostly see
the word ‘texas’ itself and a lot of punctuation and stopwords. Also,
the colors are boring!

Let’s change this:

``` r
library(RColorBrewer)
dfm_tweets %>% 
  dfm_remove(min_nchar=2) %>%
  dfm_remove("texas") %>%
  dfm_remove(stopwords()) %>%
  textplot_wordcloud(max_words=200, color=brewer.pal(8, "Dark2"),
                     random_color=TRUE, random_order = TRUE)
```

Note that this code keeps the dfm object in memory by assigning it a new
name (`dfm_tweets`), which is often useful for further analysis:

To understand more of what you are doing, look up the documentation for
the various commands (`tokens`, `dfm_remove`, `textplot_wordcloud`,
`brewer.pal` etc) in the Help pane (bottom right). For more information,
also see chapter 10 of [Computational Analysis of
Communication](https://cssbook.net).

## Tag cloud

Let’s see if we can make a tag cloud?

``` r
library(RColorBrewer)
dfm_tweets %>% 
  dfm_keep("#*") %>%
  dfm_remove("#texas") %>%
  textplot_wordcloud(max_words=200, color=brewer.pal(8, "Dark2"),
                     random_color=TRUE, random_order = TRUE)
```

Can you change this code to remove all hashtags instead, or to only show
@mentions?

As another challenge, to make an emoji cloud instead, replace
`dfm_keep("#*")` by `dfm_keep("^\\p{emoji}$", valuetype="regex")`. For
more information on regular expressions, see chapter 9 of [Computational
Analysis of Communication](https://cssbook.net)

# Corpus comparison

As a final exercise, let’s compare words used in tweets mentioning
‘texas’ to those mentioning ‘florida’. First, we obtain the second set
of tweets:

``` r
tweets_florida = search_tweets("florida", n=1000)
tweets_florida = select(tweets_florida, status_id, created_at, screen_name, lang, location, retweet_count, text)
```

We can now combine the two sets into a single dfm, and then use the

``` r
library(quanteda.textstats)
keyness = bind_rows(tweets_florida, tweets) %>% 
  corpus() %>% 
  tokens(remove_punct=TRUE) %>% 
  dfm() %>%
  textstat_keyness(target=1:nrow(tweets_florida)) 
head(keyness)
tail(keyness)
```

This can also be shown with a keyness chart:

``` r
textplot_keyness(keyness)
```
