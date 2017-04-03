DATA607\_week9\_API
================
Yun Mai
April 2, 2017

    install.packages(c("digest", "evaluate", "formatR", "highlight", "knitr", 
      "parser", "plyr", "Rcpp", "stringr"))
    install.packages(c("httr", "RCurl", "XML", "dplyr", "data.table"))
    install.packages("rmarkdown", repos = "https://cran.revolutionanalytics.com")
    install.packages("ggplot2")

Web APIs
--------

Assignment: The New York Times web site provides a rich set of APIs, as described here: <http://developer.nytimes.com/docs>

You'll need to start by signing up for an API key.

Your task is to choose one of the New York Times APIs, construct an interface in R to read in the JSON data, and transform it to an R dataframe.

Load packages

``` r
library(httr)
```

    ## Warning: package 'httr' was built under R version 3.3.3

``` r
library(RCurl)
```

    ## Warning: package 'RCurl' was built under R version 3.3.3

    ## Loading required package: bitops

``` r
library(XML)
```

    ## Warning: package 'XML' was built under R version 3.3.3

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 3.3.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(stringr)
```

    ## Warning: package 'stringr' was built under R version 3.3.3

``` r
library(knitr)
```

    ## Warning: package 'knitr' was built under R version 3.3.3

``` r
library(jsonlite)
```

    ## Warning: package 'jsonlite' was built under R version 3.3.3

``` r
library(data.table)
```

    ## -------------------------------------------------------------------------

    ## data.table + dplyr code now lives in dtplyr.
    ## Please library(dtplyr)!

    ## -------------------------------------------------------------------------

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 3.3.3

**The New York Times Developer Network**

All the APIs Fit to POST

You already know that NYTimes.com is an unparalleled source of news and information. But now it's a premier source of data, too - why just read the news when you can hack it? I will use New York Times TOP Stories API to create a interface in R.

**Top Stories API**

The Top Stories API returns lists of articles and associated images that are currently on the specified section front. The API supports JSON and JSONP.

Note: In this document, curly braces { } indicate required items. Square brackets \[ \] indicate optional items or placeholders.

To use the Top Stories API, you must sign up for an API key.

Usage is limited to 1,000 requests per day (rate limits are subject to change),and 5 calls per second.

Sample format of an xml request: <http://api.nytimes.com/svc/topstories/v2/%7Bsection%7D>.{response-format}?api-key={your-api-key}

Parameters included in the Top Stories API xml format call are shown as following:

| Parameter       | Value                                                                                                                                                                                                                                                                 |
|:----------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| section         | The section name(home, arts, automobiles, books, business, fashion, food, health, insider, magazine, movies, national, nyregion, obituaries, opinion, politics, realestate, science, sports, sundayreview, technology, theater, tmagazine, travel, upshot, and world) |
| response-format | json or jsonp                                                                                                                                                                                                                                                         |
| api-key         | Your API key                                                                                                                                                                                                                                                          |

Get top stories from the business section

``` r
url_b1 <- "http://api.nytimes.com/svc/topstories/v2/business.json"
url_b2 <- paste0(url_b1, "?api-key=", nyt_topstories_api)
response <- GET(url = url_b2)
```

The response looks like this.

    ## Response [http://api.nytimes.com/svc/topstories/v2/business.json?api-key=##]
    ##   Date: 2017-04-03 02:13
    ##   Status: 200
    ##   Content-Type: application/json
    ##   Size: 86 kB

Get the JSON file

``` r
#download URL on the business section
fetch_TS_b <- getURL(url_b2)
show <- str_sub(fetch_TS_b, 1, 300) 
show
```

    ## [1] "{\"status\":\"OK\",\"copyright\":\"Copyright (c) 2017 The New York Times Company. All Rights Reserved.\",\"section\":\"business\",\"last_updated\":\"2017-04-02T22:07:34-04:00\",\"num_results\":35,\"results\":[{\"section\":\"Technology\",\"subsection\":\"\",\"title\":\"Dyson Is the Apple of Appliances (and Just as Secretive)\",\"abs"

Parse JSON data and generate a data frame for the top business stories.

``` r
# Parse the JSON data with the fromJSON function. Under the rule of jsonlite, fromJSON function should map JSOn data into a data frame. It turned out to be a list.
parsed.business <- fromJSON(fetch_TS_b)
# conver to data frame
df_p_bu <- data.frame(parsed.business)
# show all variables
colnames_df_p_bu <- colnames(df_p_bu)
# rename some of the column names
colnames_df_p_bu_n <- str_replace(colnames_df_p_bu,"results\\.","")
colnames(df_p_bu) <- colnames_df_p_bu_n
```

Subset to select the variables intrested.

``` r
vars <- c("section", "num_results", "section", "subsection", "published_date", "title", "abstract", "url", "byline", "item_type", "published_date", "short_url")
TS_business <- df_p_bu[vars]
kable(head(TS_business))
```

| section  |  num\_results| section.1 | subsection | published\_date           | title                                                                     | abstract                                                                                                                                                                    | url                                                                                                                  | byline                 | item\_type | published\_date.1         | short\_url                |
|:---------|-------------:|:----------|:-----------|:--------------------------|:--------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------|:-----------------------|:-----------|:--------------------------|:--------------------------|
| business |            35| business  |            | 2017-04-02T18:48:24-04:00 | Dyson Is the Apple of Appliances (and Just as Secretive)                  | With an eye for both design and engineering, this British consumer electronics company is aiming for  and winning  the high end of the market.                            | <https://www.nytimes.com/2017/04/02/technology/dyson-british-consumer-electronics-company.html>                      | By MARK SCOTT          | Article    | 2017-04-02T18:48:24-04:00 | <https://nyti.ms/2nPfiL4> |
| business |            35| business  |            | 2017-04-01T10:00:25-04:00 | Policyholders in Limbo After Rare Failure of Insurer                      | Its quite possible for a regulated insurer to take policyholders into the very realm of loss and uncertainty that insurance is specifically designed to avoid.             | <https://www.nytimes.com/2017/04/01/business/policyholders-in-limbo-after-rare-failure-of-insurer.html>              | By MARY WILLIAMS WALSH | Article    | 2017-04-01T10:00:25-04:00 | <https://nyti.ms/2nI1G2u> |
| business |            35| business  | Your Taxes | 2017-03-31T17:40:46-04:00 | Our Best Guidance for Filing Your Tax Return                              | How to make filing your taxes just a little bit less painful.                                                                                                               | <https://www.nytimes.com/2017/03/31/business/yourtaxes/filing-income-tax-return.html>                                | By TIM HERRERA         | Article    | 2017-03-31T17:40:46-04:00 | <https://nyti.ms/2nIMOTa> |
| business |            35| business  |            | 2017-03-31T09:52:59-04:00 | I.R.S. Extension Adds Time to File, but You Still Have to Pay             | Taxpayers who think they will not complete their returns by the April 18 deadline may want to consider requesting an extension.                                             | <https://www.nytimes.com/2017/03/31/your-money/irs-extension-adds-time-to-file-but-you-still-have-to-pay.html>       | By ANN CARRNS          | Article    | 2017-03-31T09:52:59-04:00 | <https://nyti.ms/2nDrvRh> |
| business |            35| business  | DealBook   | 2017-04-02T14:24:38-04:00 | Facebook Pushes Outside Law Firms to Become More Diverse                  | The social media giant, like other corporations, is pressing its outside law firms to have more minorities and women working on its legal matters.                          | <https://www.nytimes.com/2017/04/02/business/dealbook/facebook-pushes-outside-law-firms-to-become-more-diverse.html> | By ELLEN ROSEN         | Article    | 2017-04-02T14:24:38-04:00 | <https://nyti.ms/2nLMzFA> |
| business |            35| business  | Media      | 2017-04-02T14:25:52-04:00 | The Ad Feels a Bit Like Oscar Bait, but Its Trying to Sell You an iPhone | Artists and brands are collaborating on advertisements that move away from mere product placement, a practice so common that the Tribeca Film Festival has an award for it. | <https://www.nytimes.com/2017/04/02/business/media/ads-short-films-shot-on-an-iphone.html>                           | By SAPNA MAHESHWARI    | Article    | 2017-04-02T14:25:52-04:00 | <https://nyti.ms/2oqjPEX> |

Get top stories from the science section

``` r
url_s1 <- "http://api.nytimes.com/svc/topstories/v2/science.json"
url_s2 <- paste0(url_s1, "?api-key=", nyt_topstories_api)

#download URL on the science section
fetch_TS_s <- getURL(url_s2)

#Parse the JSON data
parsed.science <- fromJSON(fetch_TS_s)
# conver to data frame
df_p_sc <- data.frame(parsed.science)
# show all variables
colnames_df_p_sc <- colnames(df_p_sc)
# rename some of the column names
colnames_df_p_sc_n <- str_replace(colnames_df_p_sc,"results\\.","")
colnames(df_p_sc) <- colnames_df_p_sc_n

#subset the variables interested
TS_science <- df_p_sc[vars]
kable(head(TS_science))
```

| section |  num\_results| section.1 | subsection | published\_date           | title                                                         | abstract                                                                                                                                                     | url                                                                                                     | byline                | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-----------|:--------------------------|:--------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------|:----------------------|:-----------|:--------------------------|:--------------------------|
| science |            27| science   |            | 2017-03-31T05:00:04-04:00 | In Polands Crooked Forest, a Mystery With No Straight Answer | Hundreds of pine trees outside of Gryfino, Poland, have a strange bend, and all point to the north. No one knows for certain why.                            | <https://www.nytimes.com/2017/03/31/science/crooked-forest-poland-theories.html>                        | By JOANNA KLEIN       | Article    | 2017-03-31T05:00:04-04:00 | <https://nyti.ms/2nGwBOm> |
| science |            27| science   |            | 2017-03-31T14:13:11-04:00 | First Clear View of a One-Celled Harpooner in Action          | A one-celled creature has a biological harpoon gun of remarkable complexity.                                                                                 | <https://www.nytimes.com/2017/03/31/science/dinoflagellate-video-one-celled-harpooner.html>             | By JAMES GORMAN       | Article    | 2017-03-31T14:13:11-04:00 | <https://nyti.ms/2nI2RAC> |
| science |            27| science   |            | 2017-03-31T17:42:23-04:00 | A Mysterious Flash From a Faraway Galaxy                      | Astronomers are puzzled by X-rays that for a brief time were a thousand times brighter than all of its home galaxys light.                                  | <https://www.nytimes.com/2017/03/31/science/x-ray-burst-outer-space.html>                               | By DENNIS OVERBYE     | Article    | 2017-03-31T17:42:23-04:00 | <https://nyti.ms/2nID1wG> |
| science |            27| science   |            | 2017-03-31T13:59:21-04:00 | Unmasking the Fearsome Face of a Tyrannosaur                  | A newly discovered relative to T. rex shows the dinosaurs family had a scaly face similar to a crocodiles, had no lips and a sensitive snout.              | <https://www.nytimes.com/2017/03/31/science/tyrannosaurs-face-dinosaur.html>                            | By NICHOLAS ST. FLEUR | Article    | 2017-03-31T13:59:21-04:00 | <https://nyti.ms/2nEzCxf> |
| science |            27| science   |            | 2017-03-30T18:42:24-04:00 | SpaceX Launches a Satellite With a Partly Used Rocket         | The use of a rocket booster that had flown once before may open an era of cheaper space travel, particularly for business ventures like satellite companies. | <https://www.nytimes.com/2017/03/30/science/spacex-launches-a-satellite-with-a-partly-used-rocket.html> | By KENNETH CHANG      | Article    | 2017-03-30T18:42:24-04:00 | <https://nyti.ms/2oEb1rE> |
| science |            27| science   |            | 2017-03-30T15:14:44-04:00 | Little Tropical Fish With a Big, Venomous Bite                | With their large lower canines, fang blennies deliver opioid-laced venom that seems to cause a sudden drop in their predators blood pressure.               | <https://www.nytimes.com/2017/03/30/science/fanged-blennies-fish-opioid-venom.html>                     | By STEPH YIN          | Article    | 2017-03-30T15:14:44-04:00 | <https://nyti.ms/2oDHbTT> |

Get top stories from the technology section

``` r
url_t1 <- "http://api.nytimes.com/svc/topstories/v2/technology.json"
url_t2 <- paste0(url_t1, "?api-key=", nyt_topstories_api)

#download URL on the technology section
fetch_TS_t <- getURL(url_t2)

#Parse the JSON data
parsed.technology <- fromJSON(fetch_TS_t)
# conver to data frame
df_p_te <- data.frame(parsed.technology)
# show all variables
colnames_df_p_te <- colnames(df_p_te)
# rename some of the column names
colnames_df_p_te_n <- str_replace(colnames_df_p_te,"results\\.","")
colnames(df_p_te) <- colnames_df_p_te_n

#subset the variables interested
TS_technology <- df_p_te[vars]
kable(head(TS_technology))
```

| section    |  num\_results| section.1  | subsection    | published\_date           | title                                                                       | abstract                                                                                                                                           | url                                                                                                                           | byline                                        | item\_type | published\_date.1         | short\_url                |
|:-----------|-------------:|:-----------|:--------------|:--------------------------|:----------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------|:-----------|:--------------------------|:--------------------------|
| technology |            36| technology |               | 2017-04-02T18:48:24-04:00 | Dyson Is the Apple of Appliances (and Just as Secretive)                    | With an eye for both design and engineering, this British consumer electronics company is aiming for  and winning  the high end of the market.   | <https://www.nytimes.com/2017/04/02/technology/dyson-british-consumer-electronics-company.html>                               | By MARK SCOTT                                 | Article    | 2017-04-02T18:48:24-04:00 | <https://nyti.ms/2nPfiL4> |
| technology |            36| technology |               | 2017-03-28T05:00:03-04:00 | None of Us Are Safe From Getting Owned                                    | Online, where information is power, were all at risk of being exposed as vulnerable, ignorant or worse.                                           | <https://www.nytimes.com/2017/03/28/magazine/none-of-us-are-safe-from-getting-owned.html>                                     | By AMANDA HESS                                | Article    | 2017-03-28T05:00:03-04:00 | <https://nyti.ms/2nH5K54> |
| technology |            36| technology | DealBook      | 2017-04-02T14:24:38-04:00 | Facebook Pushes Outside Law Firms to Become More Diverse                    | The social media giant, like other corporations, is pressing its outside law firms to have more minorities and women working on its legal matters. | <https://www.nytimes.com/2017/04/02/business/dealbook/facebook-pushes-outside-law-firms-to-become-more-diverse.html>          | By ELLEN ROSEN                                | Article    | 2017-04-02T14:24:38-04:00 | <https://nyti.ms/2nLMzFA> |
| technology |            36| technology | Sunday Review | 2017-04-01T14:30:19-04:00 | Jerks and the Start-Ups They Ruin                                           | Bro C.E.O.s like the head of Uber will keep destroying companies until people stop paying them.                                                    | <https://www.nytimes.com/2017/04/01/opinion/sunday/jerks-and-the-start-ups-they-ruin.html>                                    | By DAN LYONS                                  | Article    | 2017-04-01T14:30:19-04:00 | <https://nyti.ms/2nIT3Vy> |
| technology |            36| technology | Sunday Review | 2017-04-01T14:30:03-04:00 | Video Games Arent Addictive                                                | Playing them is normal behavior that at worst is a waste of time.                                                                                  | <https://www.nytimes.com/2017/04/01/opinion/sunday/video-games-arent-addictive.html>                                          | By CHRISTOPHER J. FERGUSON and PATRICK MARKEY | Article    | 2017-04-01T14:30:03-04:00 | <https://nyti.ms/2nIPnmC> |
| technology |            36| technology |               | 2017-04-01T09:00:03-04:00 | Farhads and Mikes Week in Tech: When Twitter Confuses and Facebook Copies | People always bad-mouth an update on Twitter, but this time it really was bad.                                                                     | <https://www.nytimes.com/2017/04/01/technology/farhads-and-mikes-week-in-tech-when-twitter-confuses-and-facebook-copies.html> | By FARHAD MANJOO and MIKE ISAAC               | Article    | 2017-04-01T09:00:03-04:00 | <https://nyti.ms/2om1cSD> |

Get top stories from the health section

``` r
url_h1 <- "http://api.nytimes.com/svc/topstories/v2/health.json"
url_h2 <- paste0(url_h1, "?api-key=", nyt_topstories_api)

#download URL on the health section
fetch_TS_h <- getURL(url_h2)

#Parse the JSON data
parsed.health  <- fromJSON(fetch_TS_h)
# conver to data frame
df_p_he <- data.frame(parsed.health)
# show all variables
colnames_df_p_he <- colnames(df_p_he)
# rename some of the column names
colnames_df_p_he_n <- str_replace(colnames_df_p_he,"results\\.","")
colnames(df_p_he) <- colnames_df_p_he_n

#subset the variables interested
TS_health <- df_p_he[vars]
kable(head(TS_health))
```

| section |  num\_results| section.1 | subsection | published\_date           | title                                                                          | abstract                                                                                                                                           | url                                                                                                 | byline              | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-----------|:--------------------------|:-------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------|:--------------------|:-----------|:--------------------------|:--------------------------|
| health  |            33| health    |            | 2017-03-29T09:21:08-04:00 | F.D.A. Nominee, Paid Millions by Industry, Says Hell Recuse Himself if Needed | Scott Gottlieb made millions of dollars doing work for more than 20 health care companies in the private sector.                                   | <https://www.nytimes.com/2017/03/29/health/fda-nominee-scott-gottlieb-recuse-conflicts.html>        | By KATIE THOMAS     | Article    | 2017-03-29T09:21:08-04:00 | <https://nyti.ms/2nzXWBU> |
| health  |            33| health    |            | 2017-03-30T06:00:16-04:00 | Do DHA Supplements Make Babies Smarter?                                        | Adding DHA, an omega-3 fatty acid, to infant formula or prenatal supplements doesnt improve babies brain development, recent reviews have found. | <https://www.nytimes.com/2017/03/30/well/do-dha-supplements-make-babies-smarter.html>               | By ALICE CALLAHAN   | Article    | 2017-03-30T06:00:16-04:00 | <https://nyti.ms/2oBtMMa> |
| health  |            33| health    |            | 2017-03-28T22:24:22-04:00 | F.D.A. Approves First Drug to Treat Severe Multiple Sclerosis                  | The drug, Ocrevus by Genentech, can also be used to treat patients with the more common form of the disease.                                       | <https://www.nytimes.com/2017/03/28/health/fda-drug-approved-multiple-sclerosis-ocrevus.html>       | By KATIE THOMAS     | Article    | 2017-03-28T22:24:22-04:00 | <https://nyti.ms/2oweBnF> |
| health  |            33| health    |            | 2017-03-29T06:15:03-04:00 | Therapists Offer Strategies for Postelection Stress                            | How to follow the news in a political age of anxiety.                                                                                              | <https://www.nytimes.com/2017/03/29/well/how-to-follow-the-news-in-a-political-age-of-anxiety.html> | By LESLEY ALDERMAN  | Article    | 2017-03-29T06:15:03-04:00 | <https://nyti.ms/2oxdyUf> |
| health  |            33| health    | Live       | 2017-03-31T06:00:01-04:00 | Is It Harder to Lose Weight When Youre Older?                                 | Several factors make it harder to lose weight with age.                                                                                            | <https://www.nytimes.com/2017/03/31/well/live/is-it-harder-to-lose-weight-when-youre-older.html>    | By KAREN WEINTRAUB  | Article    | 2017-03-31T06:00:01-04:00 | <https://nyti.ms/2nCKGLf> |
| health  |            33| health    |            | 2017-03-30T11:21:26-04:00 | Meet Evatar: The Lab Model That Mimics the Female Reproductive System          | Researchers hope the model, fashioned from human and mouse tissue, will help with research into endometriosis, fibroids, cancer and infertility.   | <https://www.nytimes.com/2017/03/30/science/menstrual-cycle-in-a-dish.html>                         | By CHRISTINE HAUSER | Article    | 2017-03-30T11:21:26-04:00 | <https://nyti.ms/2nDMIMU> |

Get top stories from the arts section

``` r
url_a1 <- "http://api.nytimes.com/svc/topstories/v2/arts.json"
url_a2 <- paste0(url_a1, "?api-key=", nyt_topstories_api)

#download URL on the technology section
fetch_TS_a <- getURL(url_a2)

#Parse the JSON data
parsed.arts <- fromJSON(fetch_TS_a)
# conver to data frame
df_p_ar <- data.frame(parsed.arts)
# show all variables
colnames_df_p_ar <- colnames(df_p_ar)
# rename some of the column names
colnames_df_p_ar_n <- str_replace(colnames_df_p_ar,"results\\.","")
colnames(df_p_ar) <- colnames_df_p_ar_n

#subset the variables interested
TS_arts <- df_p_ar[vars]
kable(head(TS_arts))
```

| section |  num\_results| section.1 | subsection   | published\_date           | title                                                                      | abstract                                                                                                                                              | url                                                                                                                       | byline                  | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-------------|:--------------------------|:---------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------|:------------------------|:-----------|:--------------------------|:--------------------------|
| arts    |            47| arts      | Art & Design | 2017-04-02T18:32:49-04:00 | A Hushed Departure at the Met Museum Reveals Entrenched Management Culture | The museum formerly concentrated power and information in the hands of a few but is vowing to change.                                                 | <https://www.nytimes.com/2017/04/02/arts/design/met-museum-campbell-resignation-brodsky-coburn.html>                      | By ROBIN POGREBIN       | Article    | 2017-04-02T18:32:49-04:00 | <https://nyti.ms/2nPhYIM> |
| arts    |            47| arts      | Art & Design | 2017-04-02T17:22:52-04:00 | No License Plates Here: Using Art to Transcend Prison Walls                | A mural class is part of an initiative by the State of California to bring the arts to all 35 of its adult prisons.                                   | <https://www.nytimes.com/2017/04/02/arts/design/california-prison-arts.html>                                              | By PATRICIA LEIGH BROWN | Article    | 2017-04-02T17:22:52-04:00 | <https://nyti.ms/2nM4EmQ> |
| arts    |            47| arts      | Music        | 2017-04-02T17:02:24-04:00 | Review: John Adamss Gospel Displays an Orchestral Wizards Tuneful Ear  | The St. Louis Symphonys performance on Friday at Carnegie Hall was part of the celebrations of Mr. Adamss 70th birthday year.                       | <https://www.nytimes.com/2017/04/02/arts/music/review-john-adamss-gospel-displays-an-orchestral-wizards-tuneful-ear.html> | By ZACHARY WOOLFE       | Article    | 2017-04-02T17:02:24-04:00 | <https://nyti.ms/2nLM12w> |
| arts    |            47| arts      | Music        | 2017-04-02T17:18:33-04:00 | N.E.A. to Honor Jazz Masters Under a Cloud of Uncertainty                  | Five luminaries will be honored Monday in Washington, but President Trumps proposal to scrap the National Endowment for the Arts may be distracting. | <https://www.nytimes.com/2017/04/02/arts/music/nea-jazz-masters-trump-funding.html>                                       | By GIOVANNI RUSSONELLO  | Article    | 2017-04-02T17:18:33-04:00 | <https://nyti.ms/2nLO3ja> |
| arts    |            47| arts      | Music        | 2017-04-02T16:58:56-04:00 | Review: Diving Into the Lake for a Respighi Rarity at City Opera           | La Campana Sommersa, first performed in 1927, is a tale of what goes wrong when a bell maker ventures into a fairy realm.                           | <https://www.nytimes.com/2017/04/02/arts/music/review-diving-into-the-lake-for-a-respighi-rarity-at-city-opera.html>      | By ANTHONY TOMMASINI    | Article    | 2017-04-02T16:58:56-04:00 | <https://nyti.ms/2oqISrq> |
| arts    |            47| arts      |              | 2017-04-02T16:55:04-04:00 | Now Batting: 14 New Baseball Books                                         | Publishers have filled out this springs lineup of biographies, team histories and other scholarship about the national pastime.                      | <https://www.nytimes.com/2017/04/02/books/now-batting-14-new-baseball-books.html>                                         | By DANIEL M. GOLD       | Article    | 2017-04-02T16:55:04-04:00 | <https://nyti.ms/2oqV9fD> |

Get top stories from the politics section

``` r
url_p1 <- "http://api.nytimes.com/svc/topstories/v2/politics.json"
url_p2 <- paste0(url_p1, "?api-key=", nyt_topstories_api)

#download URL on the politics section
fetch_TS_p <- getURL(url_p2)

#Parse the JSON data
parsed.politics <- fromJSON(fetch_TS_p)
# conver to data frame
df_p_po <- data.frame(parsed.politics)
# show all variables
colnames_df_p_po <- colnames(df_p_po)
# rename some of the column names
colnames_df_p_po_n <- str_replace(colnames_df_p_po,"results\\.","")
colnames(df_p_po) <- colnames_df_p_po_n

#subset the variables interested
TS_politics <- df_p_po[vars]
kable(head(TS_politics))
```

| section  |  num\_results| section.1 | subsection | published\_date           | title                                                               | abstract                                                                                                                                                                | url                                                                                                                        | byline                                          | item\_type | published\_date.1         | short\_url                |
|:---------|-------------:|:----------|:-----------|:--------------------------|:--------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------|:-----------|:--------------------------|:--------------------------|
| politics |            25| politics  | Politics   | 2017-04-02T05:39:01-04:00 | China Learns How to Get Trumps Ear: Through Jared Kushner          | Mr. Kushners role reflects the highly personal and bluntly transactional relationship between the United States and China, a risky strategy, experts say.              | <https://www.nytimes.com/2017/04/02/us/politics/trump-china-jared-kushner.html>                                            | By MARK LANDLER                                 | Article    | 2017-04-02T05:39:01-04:00 | <https://nyti.ms/2op39xP> |
| politics |            25| politics  | Politics   | 2017-04-02T21:02:56-04:00 | Trump Aides Disclosures Reveal Surge in Lucrative Political Work   | As cash has flooded Washington from a variety of groups, even the anti-establishment activists and operatives who sided with President Trump have been enriched.        | <https://www.nytimes.com/2017/04/02/us/politics/trump-aides-disclosures-reveal-explosion-in-lucrative-political-work.html> | By STEVE EDER, ERIC LIPTON and ANDREW W. LEHREN | Article    | 2017-04-02T21:02:56-04:00 | <https://nyti.ms/2nPK6eT> |
| politics |            25| politics  | Politics   | 2017-04-02T20:49:16-04:00 | Gorsuch Supreme Court Nomination Gains More Democratic Support      | Joe Donnelly of Indiana is the third Senate Democrat to back Neil M. Gorsuch, but Republicans are still five Democratic votes short of breaking any filibuster.         | <https://www.nytimes.com/2017/04/02/us/politics/gorsuch-supreme-court-democrats.html>                                      | By MICHAEL S. SCHMIDT and NOAH WEILAND          | Article    | 2017-04-02T20:49:16-04:00 | <https://nyti.ms/2nPOz1b> |
| politics |            25| politics  | Politics   | 2017-04-02T15:59:14-04:00 | In Ohio County That Backed Trump, Word of Housing Cuts Stirs Fear   | Some people in Trumbull County say they will suffer if home-repair, rental-assistance and other programs are eliminated as President Trump has sought.                  | <https://www.nytimes.com/2017/04/02/us/politics/trump-housing-budget-cuts.html>                                            | By YAMICHE ALCINDOR                             | Article    | 2017-04-02T15:59:14-04:00 | <https://nyti.ms/2nLV7fu> |
| politics |            25| politics  | Politics   | 2017-04-01T22:11:58-04:00 | Wealthy in the White House: President Trumps Inner Circle          | Many of Mr. Trumps advisers have sprawling assets and income sources, according to their financial disclosure forms.                                                   | <https://www.nytimes.com/2017/04/01/us/politics/white-house-wealth-cohn-kushner-spicer.html>                               | By ELI ROSENBERG and ANDREW W. LEHREN           | Article    | 2017-04-01T22:11:58-04:00 | <https://nyti.ms/2nJIG3H> |
| politics |            25| politics  | Politics   | 2017-04-01T19:44:55-04:00 | Michael Flynn Failed to Disclose Income From Russia-Linked Entities | The former national security adviser initially did not list income from companies linked to Russia on a financial disclosure form released by the Trump administration. | <https://www.nytimes.com/2017/04/01/us/politics/michael-flynn-financial-disclosure-russia-linked-entities.html>            | By MATTHEW ROSENBERG                            | Article    | 2017-04-01T19:44:55-04:00 | <https://nyti.ms/2nJFGoe> |

Get top stories from the world section

``` r
url_w1 <- "http://api.nytimes.com/svc/topstories/v2/world.json"
url_w2 <- paste0(url_w1, "?api-key=", nyt_topstories_api)

#download URL on the world section
fetch_TS_w <- getURL(url_w2)

#Parse the JSON data
parsed.world <- fromJSON(fetch_TS_w)
# conver to data frame
df_p_wo <- data.frame(parsed.world)
# show all variables
colnames_df_p_wo <- colnames(df_p_wo)
# rename some of the column names
colnames_df_p_wo_n <- str_replace(colnames_df_p_wo,"results\\.","")
colnames(df_p_wo) <- colnames_df_p_wo_n

#subset the variables interested
TS_world <- df_p_wo[vars]
kable(head(TS_world))
```

| section |  num\_results| section.1 | subsection   | published\_date           | title                                                               | abstract                                                                                                                                                                   | url                                                                                                | byline                           | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-------------|:--------------------------|:--------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------|:---------------------------------|:-----------|:--------------------------|:--------------------------|
| world   |            11| world     | Canada       | 2017-04-02T15:43:13-04:00 | 2 Survivors of Canadas First Quintuplet Clan Reluctantly Re-emerge | Separated from their family and exhibited in a human zoo during the 1930s, the last sisters face a new indignity: the sale of their childhood home.                        | <https://www.nytimes.com/2017/04/02/world/canada/ontario-dionne-quintuplets.html>                  | By IAN AUSTEN                    | Article    | 2017-04-02T15:43:13-04:00 | <https://nyti.ms/2oqmQ8s> |
| world   |            11| world     | Asia Pacific | 2017-04-02T12:21:25-04:00 | 20 Are Hacked and Beaten to Death at Pakistani Shrine               | The Sufi shrines custodian was arrested after followers of a self-described mystic were given an intoxicating drink and then massacred, the authorities said.             | <https://www.nytimes.com/2017/04/02/world/asia/pakistan-shrine-massacre.html>                      | By SALMAN MASOOD                 | Article    | 2017-04-02T12:21:25-04:00 | <https://nyti.ms/2nLiBRU> |
| world   |            11| world     | Americas     | 2017-04-02T13:59:29-04:00 | Rescuers and Relatives Race to Find Survivors of Colombia Mudslide  | More than 1,500 emergency workers descended on the provincial city of Mocoa after a sudden downpour sent deadly surge of mud and water through the area, killing over 200. | <https://www.nytimes.com/2017/04/02/world/americas/colombia-mudslide-survivors.html>               | By SUSAN ABAD and NICHOLAS CASEY | Article    | 2017-04-02T13:59:29-04:00 | <https://nyti.ms/2oq90CI> |
| world   |            11| world     | Asia Pacific | 2017-04-02T17:06:14-04:00 | Cambodia Appeals to Trump to Forgive War-Era Debt                   | Cambodia says the United States owes it a moral debt for the devastation it caused. Washington says a loan is a loan.                                                      | <https://www.nytimes.com/2017/04/02/world/asia/cambodia-trump-debt.html>                           | By JULIA WALLACE                 | Article    | 2017-04-02T17:06:14-04:00 | <https://nyti.ms/2nM2zr2> |
| world   |            11| world     | Europe       | 2017-04-02T18:58:19-04:00 | Thousands March in Support of Soros-Founded University in Budapest  | Hungarys prime minister is an ideological foe of the billionaire philanthropist, and a draft law is thought to target the school he opened in 1991.                       | <https://www.nytimes.com/2017/04/02/world/europe/hungary-george-soros-viktor-orban-protest.html>   | By THE ASSOCIATED PRESS          | Article    | 2017-04-02T18:58:19-04:00 | <https://nyti.ms/2nM8pbV> |
| world   |            11| world     | Asia Pacific | 2017-04-02T18:33:42-04:00 | Pakistan Approves Military Hero to Head Tricky Saudi-Led Alliance   | The professed aim is to counter Islamic extremism, but critics worry the Sunni-dominant coalition could heighten divisions or takes sides in places like Yemen.            | <https://www.nytimes.com/2017/04/02/world/asia/pakistan-general-saudi-alliance-raheel-sharif.html> | By SALMAN MASOOD and BEN HUBBARD | Article    | 2017-04-02T18:33:42-04:00 | <https://nyti.ms/2nP8ld3> |

Get top stories from the national section

``` r
url_n1 <- "http://api.nytimes.com/svc/topstories/v2/national.json"
url_n2 <- paste0(url_n1, "?api-key=", nyt_topstories_api)

#download URL on the national section
fetch_TS_n <- getURL(url_n2)

#Parse the JSON data
parsed.national <- fromJSON(fetch_TS_n)
# conver to data frame
df_p_na <- data.frame(parsed.national)
# show all variables
colnames_df_p_na <- colnames(df_p_na)
# rename some of the column names
colnames_df_p_na_n <- str_replace(colnames_df_p_na,"results\\.","")
colnames(df_p_na) <- colnames_df_p_na_n

#subset the variables interested
vars_na <- c("section", "last_updated", "num_results", "section", "subsection", "title", "abstract", "url", "byline", "item_type", "published_date", "short_url")
TS_national <- df_p_na[vars_na]
kable(head(TS_national))
```

| section  | last\_updated             |  num\_results| section.1 | subsection | title                                                                         | abstract                                                                                                                                                                                      | url                                                                                    | byline                            | item\_type | published\_date           | short\_url                |
|:---------|:--------------------------|-------------:|:----------|:-----------|:------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------|:----------------------------------|:-----------|:--------------------------|:--------------------------|
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Tribes That Live Off Coal Hold Tight to Trumps Promises                      | The Crow of Montana are among several Indian nations looking to the presidents pledges to end Obama-era coal rules or approve new oil and gas wells.                                         | <https://www.nytimes.com/2017/04/01/us/trump-coal-promises.html>                       | By JULIE TURKEWITZ                | Article    | 2017-04-01T12:18:45-04:00 | <https://nyti.ms/2nItK60> |
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Deadline Up, Families Remain in Lead-Contaminated Housing in Indiana          | The failure to evacuate the families points to several problems, like limited rental options and landlords who will not accept public housing vouchers.                                       | <https://www.nytimes.com/2017/04/01/us/west-calumet-housing-complex-lead-indiana.html> | By THE ASSOCIATED PRESS           | Article    | 2017-04-01T11:31:56-04:00 | <https://nyti.ms/2omrlka> |
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Opinion: After a Historic March, Whats Next for Women?                       | Tina Brown says the womens movement still has a lot of momentum and may return to the broader purposes of its earlier days: family and economic issues.                                      | <https://www.nytimes.com/2017/03/31/us/tina-brown-whats-next-for-women.html>           | By TINA BROWN                     | Article    | 2017-03-31T16:55:46-04:00 | <https://nyti.ms/2nF3RE4> |
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Trump University Suit Settlement Approved by Judge                            | The approval of the settlement, assuming it stands, brings to a close a case that garnered outsize national attention during the presidential race.                                           | <https://www.nytimes.com/2017/03/31/us/trump-university-settlement.html>               | By STEVE EDER and JENNIFER MEDINA | Article    | 2017-03-31T12:33:16-04:00 | <https://nyti.ms/2nE45LI> |
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Ive Got Thick Skin: We Talk to the Pro-Trump Mayor Who Was Running From Us | Things got complicated after Mayor Roger Claar of Bolingbrook, Ill., helped throw a fund-raiser for Donald J. Trump last fall. Mr. Claar hadnt answered our interview requests until Friday. | <https://www.nytimes.com/2017/03/31/us/trump-local-elections.html>                     | By JULIE BOSMAN                   | Article    | 2017-03-31T12:03:42-04:00 | <https://nyti.ms/2nHzkr4> |
| national | 2017-04-02T22:07:11-04:00 |            22| national  |            | Arkansas to Limit Guns at Sports Events After Expanding Concealed-Carry Law   | The Southeastern Conference and other college athletic conferences in the state had urged lawmakers to make the exemption, saying the expanded law raised safety concerns.                    | <https://www.nytimes.com/2017/04/01/us/arkansas-concealed-carry-stadiums.html>         | By THE ASSOCIATED PRESS           | Article    | 2017-04-01T12:40:18-04:00 | <https://nyti.ms/2omJj5Z> |

Get top stories from the nyregion section

``` r
url_y1 <- "http://api.nytimes.com/svc/topstories/v2/nyregion.json"
url_y2 <- paste0(url_y1, "?api-key=", nyt_topstories_api)

#download URL on the nyregion section
fetch_TS_y <- getURL(url_y2)

#Parse the JSON data
parsed.nyregion <- fromJSON(fetch_TS_y)
# conver to data frame
df_p_ny <- data.frame(parsed.nyregion)
# show all variables
colnames_df_p_ny <- colnames(df_p_ny)
# rename some of the column names
colnames_df_p_ny_n <- str_replace(colnames_df_p_ny,"results\\.","")
colnames(df_p_ny) <- colnames_df_p_ny_n

#subset the variables interested
TS_nyregion <- df_p_ny[vars]
kable(head(TS_nyregion))
```

| section  |  num\_results| section.1 | subsection | published\_date           | title                                                                        | abstract                                                                                                                                                                                 | url                                                                                                                            | byline                                     | item\_type | published\_date.1         | short\_url                |
|:---------|-------------:|:----------|:-----------|:--------------------------|:-----------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------|:-----------|:--------------------------|:--------------------------|
| nyregion |            10| nyregion  |            | 2017-04-02T21:12:13-04:00 | Rikers Island Commission Unveils Plan to Shut Down Jail Complex              | With Mayor Bill de Blasio now backing the plan, Melissa Mark-Viverito, the City Council speaker, announced a 10-year goal for the departure of the last inmate.                          | <https://www.nytimes.com/2017/04/02/nyregion/rikers-island-jail-closure-plan.html>                                             | By NICK CORASANITI                         | Article    | 2017-04-02T21:12:13-04:00 | <https://nyti.ms/2nPxGDQ> |
| nyregion |            10| nyregion  |            | 2017-04-02T18:39:21-04:00 | First New Ferry Arrives in New York, After Detour in Alligator Country       | The boat ended a 13-day trip from Alabama in Jersey City, in another step toward Mayor Bill de Blasios goal of opening public ferry routes this summer.                                 | <https://www.nytimes.com/2017/04/02/nyregion/new-york-ferry-bill-de-blasio.html>                                               | By PATRICK McGEEHAN                        | Article    | 2017-04-02T18:39:21-04:00 | <https://nyti.ms/2nPjKK5> |
| nyregion |            10| nyregion  |            | 2017-04-02T12:15:03-04:00 | Changes in Policing Take Hold in One of the Nations Most Dangerous Cities   | Its a sort of Hippocratic ethos: Minimize harm, and try to save lives. And in Camden, N.J., residents are noticing the results.                                                         | <https://www.nytimes.com/2017/04/02/nyregion/camden-nj-police-shootings.html>                                                  | By JOSEPH GOLDSTEIN                        | Article    | 2017-04-02T12:15:03-04:00 | <https://nyti.ms/2oq1zM3> |
| nyregion |            10| nyregion  |            | 2017-04-02T12:01:27-04:00 | Attack in Jackson Heights Leaves Two Transgender Women Living in Fear        | The friends, Nayra and Gabriela, were attacked when entering a restaurant in Jackson Heights, a neighborhood known as welcoming to gays and lesbians.                                    | <https://www.nytimes.com/2017/04/02/nyregion/a-day-out-leaves-two-transgender-women-living-in-fear.html>                       | By DAVID GONZALEZ                          | Article    | 2017-04-02T12:01:27-04:00 | <https://nyti.ms/2opZUGa> |
| nyregion |            10| nyregion  |            | 2017-04-02T16:33:42-04:00 | Legal Team That Includes Giuliani Pushes for Deal, Just Not With Prosecutors | In an Iran sanctions case, Rudolph Giuliani and Michael Mukasey have tried to negotiate a resolution to Reza Zarrabs charges with U.S. and Turkey officials, according to court papers. | <https://www.nytimes.com/2017/04/02/nyregion/sessions-notified-of-giuliani-bid-for-deal-in-gold-trader-case-filings-show.html> | By BENJAMIN WEISER and WILLIAM K. RASHBAUM | Article    | 2017-04-02T16:33:42-04:00 | <https://nyti.ms/2nLTDSo> |
| nyregion |            10| nyregion  |            | 2017-03-31T20:42:50-04:00 | Albany Works Overtime as Budget Deal Proves Elusive                          | Lawmakers continued to talk on Saturday, with stumbling blocks centered on issues like charter schools and raising the age of criminal responsibility.                                   | <https://www.nytimes.com/2017/03/31/nyregion/new-york-state-budget-andrew-cuomo.html>                                          | By JESSE McKINLEY and LISA W. FODERARO     | Article    | 2017-03-31T20:42:50-04:00 | <https://nyti.ms/2nJoT6p> |

Get top stories from the theater section

``` r
url_th1 <- "http://api.nytimes.com/svc/topstories/v2/theater.json"
url_th2 <- paste0(url_th1, "?api-key=", nyt_topstories_api)

#download URL on the theater section
fetch_TS_th <- getURL(url_th2)

#Parse the JSON data
parsed.theater <- fromJSON(fetch_TS_th)
# conver to data frame
df_p_th <- data.frame(parsed.theater)
# show all variables
colnames_df_p_th <- colnames(df_p_th)
# rename some of the column names
colnames_df_p_th_n <- str_replace(colnames_df_p_th,"results\\.","")
colnames(df_p_th) <- colnames_df_p_th_n

#subset the variables interested
TS_theater <- df_p_th[vars]
kable(head(TS_theater))
```

| section |  num\_results| section.1 | subsection | published\_date           | title                                                                  | abstract                                                                                                                                                          | url                                                                                                    | byline                   | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-----------|:--------------------------|:-----------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------|:-------------------------|:-----------|:--------------------------|:--------------------------|
| theater |            21| theater   |            | 2017-03-31T22:00:17-04:00 | Review: A Mesmerizing Hairy Ape Brings Existentialism to Park Avenue | In this visually ravishing production, Bobby Cannavale steps into a part that has been waiting for him for decades.                                               | <https://www.nytimes.com/2017/03/31/theater/hairy-ape-review-bobby-cannavale-eugene-oneill.html>       | By BEN BRANTLEY          | Article    | 2017-03-31T22:00:17-04:00 | <https://nyti.ms/2nJCE4U> |
| theater |            21| theater   |            | 2017-04-01T00:01:03-04:00 | Five Must-See Shows if Youre in New York This Month                   | Offerings include a matchmaker named Dolly (embodied by a little old diva named Bette), a musical about a shadow-spotting mammal and a new work from Annie Baker. | <https://www.nytimes.com/2017/04/01/theater/five-must-see-shows-if-youre-in-new-york-this-month.html>  | By BEN BRANTLEY          | Article    | 2017-04-01T00:01:03-04:00 | <https://nyti.ms/2nGfbQf> |
| theater |            21| theater   |            | 2017-03-30T17:43:35-04:00 | Review: Pathos Times Two: A Double Dose of Inge, in Close Quarters     | The 1950s plays Picnic and Come Back, Little Sheba, in repertory revivals at the Gym at Judson, capture the playwrights gift for understatement.             | <https://www.nytimes.com/2017/03/30/theater/picnic-come-back-little-sheba-review-transport-group.html> | By ELISABETH VINCENTELLI | Article    | 2017-03-30T17:43:35-04:00 | <https://nyti.ms/2oEaV2Z> |
| theater |            21| theater   |            | 2017-03-30T15:09:20-04:00 | Faith and Identity Clash in The Profane: An Actors Round Table      | The play, by Zayd Dohrn, finds conflict between freedom and fundamentalism in a story of a marriage between the children of Middle Eastern immigrants.            | <https://www.nytimes.com/2017/03/30/theater/the-profane-zayd-dohrn-playwrights-horizons.html>          | By ALEXIS SOLOSKI        | Article    | 2017-03-30T15:09:20-04:00 | <https://nyti.ms/2oDEJwZ> |
| theater |            21| theater   |            | 2017-03-30T17:39:36-04:00 | Review: A Warrior, Leaning on Shakespeare, in Cry Havoc!             | Stephan Wolfert describes his experiences during and after the Persian Gulf war of 1991 in this autobiographical solo show.                                       | <https://www.nytimes.com/2017/03/30/theater/cry-havoc-review.html>                                     | By ALEXIS SOLOSKI        | Article    | 2017-03-30T17:39:36-04:00 | <https://nyti.ms/2nF5raI> |
| theater |            21| theater   |            | 2017-03-29T14:41:18-04:00 | How to Get Cheap Tickets to Broadway Shows (Even Hamilton)           | From lotteries to apps, there are several easy ways to score inexpensive tickets to a show. Even one about a certain former Treasury secretary.                   | <https://www.nytimes.com/2017/03/29/theater/how-to-get-cheap-theater-tickets-including-hamilton.html>  | By ERIK PIEPENBURG       | Article    | 2017-03-29T14:41:18-04:00 | <https://nyti.ms/2oyX91y> |

Get top stories from the sports section

``` r
url_sp1 <- "http://api.nytimes.com/svc/topstories/v2/sports.json"
url_sp2 <- paste0(url_sp1, "?api-key=", nyt_topstories_api)

#download URL on the sports section
fetch_TS_sp <- getURL(url_sp2)

#Parse the JSON data
parsed.sports <- fromJSON(fetch_TS_sp)
# conver to data frame
df_p_sp <- data.frame(parsed.sports)
# show all variables
colnames_df_p_sp <- colnames(df_p_sp)
# rename some of the column names
colnames_df_p_sp_n <- str_replace(colnames_df_p_sp,"results\\.","")
colnames(df_p_sp) <- colnames_df_p_sp_n

#subset the variables interested
TS_sports <- df_p_sp[vars]
kable(head(TS_sports))
```

| section |  num\_results| section.1 | subsection         | published\_date           | title                                                                   | abstract                                                                                                                                                                          | url                                                                                                            | byline             | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-------------------|:--------------------------|:------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------|:-------------------|:-----------|:--------------------------|:--------------------------|
| sports  |            17| sports    | College Basketball | 2017-04-02T20:20:48-04:00 | South Carolina Defeats Mississippi State to Win Womens Title           | Dawn Staley became the second African-American coach to win a title since the N.C.A.A. began sponsoring a womens basketball tournament in 1982.                                  | <https://www.nytimes.com/2017/04/02/sports/ncaabasketball/south-carolina-mississippi-state-ncaa-womens.html>   | By JERÉ LONGMAN    | Article    | 2017-04-02T20:20:48-04:00 | <https://nyti.ms/2nPMKRD> |
| sports  |            17| sports    | Baseball           | 2017-04-02T17:22:47-04:00 | Yankees Stumble Out of the Gate Again and Fall to the Rays              | After the best spring training in team history, the Yankees lost on opening day for a sixth straight year.                                                                        | <https://www.nytimes.com/2017/04/02/sports/baseball/new-york-yankees-tampa-bay-rays-opening-day.html>          | By BILLY WITZ      | Article    | 2017-04-02T17:22:47-04:00 | <https://nyti.ms/2oqYqLX> |
| sports  |            17| sports    | College Basketball | 2017-04-02T07:00:17-04:00 | Gonzaga Earns Chance to Prove It Belongs Among Basketballs Elite       | Playing in a weaker conference and defeating lower-seeded teams in the N.C.A.A. tournament has done it no favors. Now comes a No. 1 seed, and an opportunity to make a statement. | <https://www.nytimes.com/2017/04/02/sports/ncaabasketball/gonzaga-final-four-north-carolina.html>              | By MARC TRACY      | Article    | 2017-04-02T07:00:17-04:00 | <https://nyti.ms/2nKJkht> |
| sports  |            17| sports    | Baseball           | 2017-04-02T19:23:21-04:00 | How Much Harder Can a Starter Throw? Syndergaard Could Find Out         | An exceptional delivery and build helped Noah Syndergaard throw the fastest average fastball last year for a major league starter: 97.9 m.p.h. He may not have peaked.            | <https://www.nytimes.com/2017/04/02/sports/baseball/noah-syndegaard-mets-fastball-velocity.html>               | By JAMES WAGNER    | Article    | 2017-04-02T19:23:21-04:00 | <https://nyti.ms/2nPnL0O> |
| sports  |            17| sports    | Hockey             | 2017-04-02T18:11:29-04:00 | One Team, 406 Goals, a Million Stories: Mr. Ranger Is Still Making Fans | Rod Gilbert, the Rangers career scoring leader, can be found at home games traversing Madison Square Garden with the same zeal with which he played years ago.                   | <https://www.nytimes.com/2017/04/02/sports/hockey/rangers-rod-gilbert.html>                                    | By ALLAN KREDA     | Article    | 2017-04-02T18:11:29-04:00 | <https://nyti.ms/2oqXUxe> |
| sports  |            17| sports    | Tennis             | 2017-04-02T20:26:01-04:00 | Roger Federer Has Found His Form. Now Hell Give the Clay a Rest.       | Federer won the Miami Open by beating Rafael Nadal for the third time in three months. To stay fresh, he plans a break before the French Open.                                    | <https://www.nytimes.com/2017/04/02/sports/tennis/roger-federer-rest-french-open-miami-clay-rafael-nadal.html> | By STEPHANIE MYLES | Article    | 2017-04-02T20:26:01-04:00 | <https://nyti.ms/2nMi95P> |

Get top stories from the sports section

``` r
url_fs1 <- "http://api.nytimes.com/svc/topstories/v2/fashion.json"
url_fs2 <- paste0(url_fs1, "?api-key=", nyt_topstories_api)

#download URL on the sports section
fetch_TS_fs <- getURL(url_fs2)

#Parse the JSON data
parsed.fashion <- fromJSON(fetch_TS_fs)
# conver to data frame
df_p_fs <- data.frame(parsed.fashion)
# show all variables
colnames_df_p_fs <- colnames(df_p_fs)
# rename some of the column names
colnames_df_p_fs_n <- str_replace(colnames_df_p_fs,"results\\.","")
colnames(df_p_fs) <- colnames_df_p_fs_n

#subset the variables interested
TS_fashion <- df_p_fs[vars]
kable(head(TS_fashion))
```

| section |  num\_results| section.1 | subsection  | published\_date           | title                                                        | abstract                                                                                                                                                                                                                             | url                                                                                                                 | byline                                    | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:------------|:--------------------------|:-------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------|:------------------------------------------|:-----------|:--------------------------|:--------------------------|
| fashion |            33| fashion   |             | 2017-04-01T06:00:28-04:00 | My Son, My Daughter: A Mothers Evolution                    | Isabel Rose, the New York real estate heiress, decided to go public about her childs gender transition after President Trump rescinded federal protections for transgender students.                                                | <https://www.nytimes.com/2017/04/01/style/isabel-rose-transgender-daughter-donald-trump.html>                       | By JACOB BERNSTEIN                        | Article    | 2017-04-01T06:00:28-04:00 | <https://nyti.ms/2olCfXk> |
| fashion |            33| fashion   |             | 2017-04-01T00:13:06-04:00 | Style on Taipeis Streets: A Brando Look and a Lot of Berets | During a month in Taiwan, the photographer An Rong Xu captured a bustling city with a rich array of looks influenced by modern-day street style, Parisian cafe culture, the American West, Marlon Brando and 1990s Hong Kong cinema. | <https://www.nytimes.com/2017/04/01/style/style-on-taipeis-streets-a-brando-look-and-a-lot-of-berets.html>          | By AN RONG XU, JOANNA NIKAS and EVE LYONS | Article    | 2017-04-01T00:13:06-04:00 | <https://nyti.ms/2nGdna7> |
| fashion |            33| fashion   |             | 2017-04-01T05:13:37-04:00 | Peretti Siblings Share a Sense of Humor, Not Just Genes      | Chelsea and Jonah Peretti spend a morning at a museum, discussing BuzzFeed, Get Out and who tells better jokes. They teased each other a lot too.                                                                                  | <https://www.nytimes.com/2017/04/01/fashion/jonah-peretti-chelsea-peretti-get-out-buzzfeed-brooklyn-nine-nine.html> | By ALEX BHATTACHARJI                      | Article    | 2017-04-01T05:13:37-04:00 | <https://nyti.ms/2olpxrz> |
| fashion |            33| fashion   |             | 2017-04-01T06:00:01-04:00 | Love Lessons From the (Very) First Couple                    | Adam and Eve, but mostly Eve, are victims of the greatest character assassination ever.                                                                                                                                              | <https://www.nytimes.com/2017/04/01/style/love-lessons-from-the-first-couple-adam-and-eve.html>                     | By BRUCE FEILER                           | Article    | 2017-04-01T06:00:01-04:00 | <https://nyti.ms/2nHbYzX> |
| fashion |            33| fashion   |             | 2017-03-31T00:00:03-04:00 | The Accident No One Talked About                             | When a family tries to sweep tragedy under the rug, the damage is deep and lasting.                                                                                                                                                  | <https://www.nytimes.com/2017/03/31/fashion/modern-love-the-accident-no-one-talked-about.html>                      | By JESSICA CIENCIN HENRIQUEZ              | Article    | 2017-03-31T00:00:03-04:00 | <https://nyti.ms/2nFVkSU> |
| fashion |            33| fashion   | Mens Style | 2017-03-31T00:00:04-04:00 | Andrew McCarthys Newest Role: Young Adult Novelist          | A burger and fries with a longtime New Yorker whose son is working on a movie with his dads Pretty in Pink co-star, Molly Ringwald.                                                                                               | <https://www.nytimes.com/2017/03/31/fashion/mens-style/andrew-mccarthy-young-adult-novel.html>                      | By MARIA RUSSO                            | Article    | 2017-03-31T00:00:04-04:00 | <https://nyti.ms/2nBQ69j> |

Get top stories from the home section

``` r
url_hm1 <- "http://api.nytimes.com/svc/topstories/v2/home.json"
url_hm2 <- paste0(url_hm1, "?api-key=", nyt_topstories_api)

#download URL on the sports section
fetch_TS_hm <- getURL(url_hm2)

#Parse the JSON data
parsed.home <- fromJSON(fetch_TS_hm)
# conver to data frame
df_p_hm <- data.frame(parsed.home)
# show all variables
colnames_df_p_hm <- colnames(df_p_hm)
# rename some of the column names
colnames_df_p_hm_n <- str_replace(colnames_df_p_hm,"results\\.","")
colnames(df_p_hm) <- colnames_df_p_hm_n

#subset the variables interested
TS_home <- df_p_hm[vars]
kable(head(TS_home))
```

| section |  num\_results| section.1 | subsection | published\_date           | title                                                               | abstract                                                                                                                                                         | url                                                                                                                        | byline                                          | item\_type | published\_date.1         | short\_url                |
|:--------|-------------:|:----------|:-----------|:--------------------------|:--------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------|:-----------|:--------------------------|:--------------------------|
| home    |            33| home      |            | 2017-04-02T13:53:43-04:00 | Xi Jinping, Ivanka Trump, Colombia: Your Morning Briefing           | Heres what you need to know to start your day.                                                                                                                  | <https://www.nytimes.com/2017/04/02/briefing/ivanka-trump-colombia-xi-jinping.html>                                        | By CHARLES McDERMID                             | Article    | 2017-04-02T13:53:43-04:00 | <https://nyti.ms/2oqoery> |
| home    |            33| home      | Politics   | 2017-04-02T05:39:01-04:00 | China Learns How to Get Trumps Ear: Through Jared Kushner          | Mr. Kushners role reflects the highly personal and bluntly transactional relationship between the United States and China, a risky strategy, experts say.       | <https://www.nytimes.com/2017/04/02/us/politics/trump-china-jared-kushner.html>                                            | By MARK LANDLER                                 | Article    | 2017-04-02T05:39:01-04:00 | <https://nyti.ms/2op39xP> |
| home    |            33| home      | Politics   | 2017-04-01T12:31:29-04:00 | Trump Couple, Now White House Employees, Cant Escape Conflict Laws | Jared Kushner and Ivanka Trump are walking on perilous legal and ethical ground, according to several prominent experts on the subject.                          | <https://www.nytimes.com/2017/04/01/us/politics/ivanka-trump-jared-kushner-conflicts-business-empire.html>                 | By ERIC LIPTON and JESSE DRUCKER                | Article    | 2017-04-01T12:31:29-04:00 | <https://nyti.ms/2omJRZU> |
| home    |            33| home      | Politics   | 2017-04-02T21:02:56-04:00 | Trump Aides Disclosures Reveal Surge in Lucrative Political Work   | As cash has flooded Washington from a variety of groups, even the anti-establishment activists and operatives who sided with President Trump have been enriched. | <https://www.nytimes.com/2017/04/02/us/politics/trump-aides-disclosures-reveal-explosion-in-lucrative-political-work.html> | By STEVE EDER, ERIC LIPTON and ANDREW W. LEHREN | Article    | 2017-04-02T21:02:56-04:00 | <https://nyti.ms/2nPK6eT> |
| home    |            33| home      | Politics   | 2017-04-01T22:11:58-04:00 | Wealthy in the White House: President Trumps Inner Circle          | Many of Mr. Trumps advisers have sprawling assets and income sources, according to their financial disclosure forms.                                            | <https://www.nytimes.com/2017/04/01/us/politics/white-house-wealth-cohn-kushner-spicer.html>                               | By ELI ROSENBERG and ANDREW W. LEHREN           | Article    | 2017-04-01T22:11:58-04:00 | <https://nyti.ms/2nJIG3H> |
| home    |            33| home      | Politics   | 2017-04-02T20:49:16-04:00 | Gorsuch Supreme Court Nomination Gains More Democratic Support      | Joe Donnelly of Indiana is the third Senate Democrat to back Neil M. Gorsuch, but Republicans are still five Democratic votes short of breaking any filibuster.  | <https://www.nytimes.com/2017/04/02/us/politics/gorsuch-supreme-court-democrats.html>                                      | By MICHAEL S. SCHMIDT and NOAH WEILAND          | Article    | 2017-04-02T20:49:16-04:00 | <https://nyti.ms/2nPOz1b> |

``` r
stat_va <- c("section", "num_results")
section_number <- rbind(TS_business[1,stat_va], TS_science[1,stat_va], TS_technology[1,stat_va], TS_health[1,stat_va], TS_politics[1,stat_va], TS_world[1,stat_va], TS_national[1,stat_va], TS_nyregion[1,stat_va], TS_arts[1,stat_va], TS_theater[1,stat_va], TS_fashion[1,stat_va], TS_sports[1,stat_va],TS_home[1,stat_va])
```

``` r
ggplot(data=section_number,aes(section, num_results, fill = section)) +
  geom_bar(stat="identity", position = "stack") +
  theme(legend.position = "none") + 
  ggtitle("Top Stories of NY Times Articles by Section") +
  xlab("Section") + ylab("Number of articles") +
  geom_text(aes(label=num_results), vjust=0.5, hjust=0.8,color="black")+
  theme(axis.text.x=element_text(angle=45, hjust=1))
```

![](DA607_homework9_APIs_files/figure-markdown_github/unnamed-chunk-22-1.png)

From the distribution of different topics of top stories, we could know the targeted audience of New Yok Times. In the 11 sections I chose, arts is New York Times's favorite topic for top story. Technology, business, health, fashion and home are five popular topics. science, politics, theater, and sports are less popular than the former 6 sections. It is not known how popular book and movie are because the data is not available, showing error type 403 when trying to get info from NYT top stories API. Local and world news contributes the least to the top stories.
