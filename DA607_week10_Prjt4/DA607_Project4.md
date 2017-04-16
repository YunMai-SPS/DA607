DA607\_week10\_Dopcument Classification
================
Yun Mai
April 14, 2017

Document Classification
-----------------------

It can be useful to be able to classify new "test" documents using already classified "training" documents. A common example is using a corpus of labeled spam and ham (non-spam) e-mails to predict whether or not a new document is spam.

For this project, you can start with a spam/ham dataset, then predict the class of new documents (either withheld from the training dataset or from another source such as your own spam folder). One example corpus: <https://spamassassin.apache.org/publiccorpus/>

    install.packages("RTextTools")
    install.packages("SnowballC")
    install.packages("vcd")
    install.packages("topicmodels")

``` r
options(repos="https://cran.rstudio.com" )
library(RCurl)
```

    ## Warning: package 'RCurl' was built under R version 3.3.3

    ## Loading required package: bitops

``` r
library(stringr)
```

    ## Warning: package 'stringr' was built under R version 3.3.3

``` r
library(tm)
```

    ## Warning: package 'tm' was built under R version 3.3.3

    ## Loading required package: NLP

``` r
library(RTextTools) 
```

    ## Warning: package 'RTextTools' was built under R version 3.3.3

    ## Loading required package: SparseM

    ## Warning: package 'SparseM' was built under R version 3.3.3

    ## 
    ## Attaching package: 'SparseM'

    ## The following object is masked from 'package:base':
    ## 
    ##     backsolve

``` r
library(SnowballC)
```

    ## 
    ## Attaching package: 'SnowballC'

    ## The following objects are masked from 'package:RTextTools':
    ## 
    ##     getStemLanguages, wordStem

``` r
library(knitr)
```

    ## Warning: package 'knitr' was built under R version 3.3.3

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 3.3.3

    ## 
    ## Attaching package: 'ggplot2'

    ## The following object is masked from 'package:NLP':
    ## 
    ##     annotate

``` r
library(tidyr)
```

    ## Warning: package 'tidyr' was built under R version 3.3.3

    ## 
    ## Attaching package: 'tidyr'

    ## The following object is masked from 'package:RCurl':
    ## 
    ##     complete

``` r
update.packages("tm",  checkBuilt = TRUE)
```

``` r
if(!file.exists("easy_ham")) dir.create("easy_ham")
download.file(url = "http://spamassassin.apache.org/old/publiccorpus/20021010_easy_ham.tar.bz2", destfile = "20021010_easy_ham.tar.bz2")
untar("20021010_easy_ham.tar.bz2")

if(!file.exists("hard_ham")) dir.create("hard_ham")
download.file(url = "http://spamassassin.apache.org/old/publiccorpus/20021010_hard_ham.tar.bz2", destfile = "20021010_hard_ham.tar.bz2")
untar("20021010_hard_ham.tar.bz2")

if(!file.exists("spam")) dir.create("spam")
download.file(url = "http://spamassassin.apache.org/old/publiccorpus/20021010_spam.tar.bz2", destfile = "20021010_spam.tar.bz2")
untar("20021010_spam.tar.bz2")
```

Document the emails in a structured table
-----------------------------------------

``` r
easy_ham <- list.files(path="easy_ham/", full.names=T, recursive=F)
hard_ham <- list.files(path="hard_ham/", full.names=T, recursive=F)
spam <- list.files(path="spam/", full.names=T, recursive=F)
```

Test: setting corpus
--------------------

``` r
tmp <- readLines(easy_ham[1])
tmp <- str_c(tmp, collapse = "")
ham.corpus <- VCorpus(VectorSource(tmp))
ham.corpus
```

    ## <<VCorpus>>
    ## Metadata:  corpus specific: 0, document level (indexed): 0
    ## Content:  documents: 1

Test: Add meta information
--------------------------

Email type (ham or spam), subject, sender will be added to the meta data.

``` r
meta(ham.corpus[[1]], "type",  type = "corpus") <- "Ham" 
meta(ham.corpus[[1]], "subject") <- unlist((str_extract_all(readLines(easy_ham[1]), "^Subject\\:.+")))
meta(ham.corpus[[1]], "From") <- unlist((str_extract_all(readLines(easy_ham[1]), "^From\\:.+")))
meta(ham.corpus[[1]], type = "corpus")
```

    ##   author       : character(0)
    ##   datetimestamp: 2017-04-16 03:50:00
    ##   description  : character(0)
    ##   heading      : character(0)
    ##   id           : 1
    ##   language     : en
    ##   origin       : character(0)
    ##   type         : Ham
    ##   subject      : Subject: Re: New Sequences Window
    ##   From         : From: Robert Elz <kre@munnari.OZ.AU>

Creat the corpus for analysis
-----------------------------

\*\* easy\_ham \*\*

``` r
n <- 1
for (i in 2:length(easy_ham)){
  tmp <- readLines(easy_ham[i])
  tmp <- str_c(tmp, collapse = "")
  # remove whitespace
  tmp <- str_trim(unlist(str_replace_all(tmp,"\\s+"," ")))   
  if (length(easy_ham)!=0) {
    tmp.corpus <- VCorpus(VectorSource(tmp))
    ham.corpus <- c(ham.corpus,tmp.corpus)
    n <- n+1
    meta(ham.corpus[[n]], "type",  type = "corpus") <- "Ham"
    meta(ham.corpus[[n]], "subject") <- unlist((str_extract_all(readLines(easy_ham[i]), "^Subject\\:.+")))
    meta(ham.corpus[[n]], "From") <- unlist((str_extract_all(readLines(easy_ham[i]), "^From\\:.+")))
    }
}
```

**add hard\_ham **

``` r
k <- length(easy_ham)
for (i in 1:length(hard_ham)){
  tmp <- readLines(hard_ham[i])
  tmp <- str_c(tmp, collapse = "")
  
  if (length(hard_ham)!=0) {
    tmp.corpus <- VCorpus(VectorSource(tmp))
    ham.corpus <- c(ham.corpus,tmp.corpus)
    k <- k+1
    meta(ham.corpus[[k]], "type",  type = "corpus") <- "Ham"
    meta(ham.corpus[[k]], "subject") <- unlist((str_extract_all(readLines(hard_ham[i]), "^Subject\\:.+")))
    meta(ham.corpus[[k]], "From") <- unlist((str_extract_all(readLines(hard_ham[i]), "^From\\:.+")))
    }
}
```

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0231.7c6cc716ce3f3bfad7130dd3c8d7b072'

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0231.7c6cc716ce3f3bfad7130dd3c8d7b072'

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0231.7c6cc716ce3f3bfad7130dd3c8d7b072'

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0250.7c6cc716ce3f3bfad7130dd3c8d7b072'

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0250.7c6cc716ce3f3bfad7130dd3c8d7b072'

    ## Warning in readLines(hard_ham[i]): incomplete final line found on
    ## 'hard_ham/0250.7c6cc716ce3f3bfad7130dd3c8d7b072'

The first file in spam folder is not a normal email file but a directory. We will need to remove this kind of abnormal file before genearte corpus. Since each email must have at least a receipient, there will be at least one string starting with "To". Abnormal file with no text starting with "To" will be not be considered as email and will be dropped.

``` r
length(spam)
```

    ## [1] 501

``` r
for (i in 1:length(spam)){
    tmp <- data.frame(readLines(spam[i]))
    if (sum(str_count(tmp[,1], "^To")) == 0 ){
      file.remove(spam[i])
    }
}
```

    ## Warning in readLines(spam[i]): incomplete final line found on 'spam/
    ## 0143.260a940290dcb61f9327b224a368d4af'

``` r
spam <- list.files(path="spam/", full.names=T, recursive=F)
length(spam)
```

    ## [1] 500

**spam **

``` r
# read all of files from spam to generate spam corpus
n <- 0
for (i in 1:length(spam)){
  tmp <- readLines(spam[i])
  tmp <- str_c(tmp, collapse = "")
  tmp.corpus <- VCorpus(VectorSource(tmp))
  ifelse (!exists('spam.corpus'), 
           spam.corpus <- tmp.corpus,
           spam.corpus <- c(spam.corpus,tmp.corpus))
    n <- n+1
    meta(spam.corpus[[n]], "type",  type = "corpus") <- "Spam"
    meta(spam.corpus[[n]], "subject") <- unlist((str_extract_all(readLines(spam[i]), "^Subject\\:.+")))
    meta(spam.corpus[[n]], "From") <- unlist((str_extract_all(readLines(spam[i]), "^From\\:.+")))
}
```

    ## Warning in readLines(spam[i]): incomplete final line found on 'spam/
    ## 0143.260a940290dcb61f9327b224a368d4af'

    ## Warning in readLines(spam[i]): incomplete final line found on 'spam/
    ## 0143.260a940290dcb61f9327b224a368d4af'

    ## Warning in readLines(spam[i]): incomplete final line found on 'spam/
    ## 0143.260a940290dcb61f9327b224a368d4af'

``` r
# combind ham and spam corpus and shuffle the files
mix.corpus <- c(ham.corpus, spam.corpus)
set.seed(123)
mix.corpus <- sample(mix.corpus, length(mix.corpus))
```

``` r
# meta data
meta(mix.corpus[[1]], type = "corpus")
```

    ##   author       : character(0)
    ##   datetimestamp: 2017-04-16 03:50:04
    ##   description  : character(0)
    ##   heading      : character(0)
    ##   id           : 1
    ##   language     : en
    ##   origin       : character(0)
    ##   type         : Ham
    ##   subject      : Subject: [IRR] Re: The 3rd Annual Consult Hyperion Digital Identity Forum
    ##   From         : From: "David G.W. Birch" <dgw-lists@birches.org>

``` r
#  text content
mix.corpus[[1]]$content
```

    ## [1] "From irregulars-admin@tb.tf Sat Oct 5 12:38:48 2002Return-Path: <irregulars-admin@tb.tf>Delivered-To: yyyy@localhost.example.comReceived: from localhost (jalapeno [127.0.0.1]) by jmason.org (Postfix) with ESMTP id 102BF16F16 for <jm@localhost>; Sat, 5 Oct 2002 12:38:48 +0100 (IST)Received: from jalapeno [127.0.0.1] by localhost with IMAP (fetchmail-5.9.0) for jm@localhost (single-drop); Sat, 05 Oct 2002 12:38:48 +0100 (IST)Received: from web.tb.tf (route-64-131-126-36.telocity.com [64.131.126.36]) by dogma.slashnull.org (8.11.6/8.11.6) with ESMTP id g94JcjK05789 for <jm-irr@jmason.org>; Fri, 4 Oct 2002 20:38:46 +0100Received: from web.tb.tf (localhost.localdomain [127.0.0.1]) by web.tb.tf (8.11.6/8.11.6) with ESMTP id g94Jm2I26430; Fri, 4 Oct 2002 15:48:03 -0400Received: from red.harvee.home (red [192.168.25.1] (may be forged)) by web.tb.tf (8.11.6/8.11.6) with ESMTP id g94JlKI26420 for <irregulars@tb.tf>; Fri, 4 Oct 2002 15:47:21 -0400Received: from texas.pobox.com (texas.pobox.com [64.49.223.111]) by red.harvee.home (8.11.6/8.11.6) with ESMTP id g94JcLD11866 for <irregulars@tb.tf>; Fri, 4 Oct 2002 15:38:22 -0400Received: from [192.168.0.4] (pc2-woki1-4-cust149.gfd.cable.ntl.com [62.254.239.149]) by texas.pobox.com (Postfix) with ESMTP id 860AE4535A; Fri, 4 Oct 2002 15:38:18 -0400 (EDT)User-Agent: Microsoft-Entourage/10.1.0.2006From: \"David G.W. Birch\" <dgw-lists@birches.org>To: <e$@vmeng.com>, <dcsb@ai.mit.edu>, <cryptography@wasabisystems.com>, <mac_crypto@vmeng.com>, <fork@xent.com>, <irregulars@tb.tf>Message-Id: <B9C3A923.4976%dgw-lists@birches.org>In-Reply-To: <p05111a5ab9c2875b09bf@[66.149.49.6]>MIME-Version: 1.0Content-Type: text/plain; charset=\"US-ASCII\"Content-Transfer-Encoding: 7bitSubject: [IRR] Re: The 3rd Annual Consult Hyperion Digital Identity ForumSender: irregulars-admin@tb.tfErrors-To: irregulars-admin@tb.tfX-Beenthere: irregulars@tb.tfX-Mailman-Version: 2.0.6Precedence: bulkList-Help: <mailto:irregulars-request@tb.tf?subject=help>List-Post: <mailto:irregulars@tb.tf>List-Subscribe: <http://tb.tf/mailman/listinfo/irregulars>, <mailto:irregulars-request@tb.tf?subject=subscribe>List-Id: New home of the TBTF Irregulars mailing list <irregulars.tb.tf>List-Unsubscribe: <http://tb.tf/mailman/listinfo/irregulars>, <mailto:irregulars-request@tb.tf?subject=unsubscribe>List-Archive: <http://tb.tf/mailman/private/irregulars/>Date: Fri, 04 Oct 2002 20:22:59 +0100X-Spam-Status: No, hits=-1.8 required=5.0 tests=IN_REP_TO,KNOWN_MAILING_LIST,T_NONSENSE_FROM_20_30, USER_AGENT,USER_AGENT_ENTOURAGE version=2.50-cvsX-Spam-Level: On 4/10/02 1:13 am, someone e-said:> The guy messed up his own URL. It should be> http://www.digitalidforum.com which redirects to> http://www.consult.hyperion.co.uk/digid3.htmlI didn't mess it up: I f*cked it up by not paying attention to acopy-and-paste from something else.Next time, I really will leave it to the PR guys.Best regards,Dave Birch._______________________________________________Irregulars mailing listIrregulars@tb.tfhttp://tb.tf/mailman/listinfo/irregulars"

    Corpuses need to be cleaned for further analysis.

Clean the data
--------------

``` r
# remove numbers, puctuation characters, stopwords, uppercase, and sparse terms and reduce words to their stem in the term-document matrix. 
mix.corpus <- tm_map(mix.corpus, content_transformer(removeNumbers)) %>% 
  tm_map(content_transformer(removePunctuation)) %>% 
  tm_map(removeWords, words = stopwords(kind = "en")) %>% 
  tm_map(content_transformer(tolower)) %>% 
  tm_map(content_transformer(stemDocument)) 
```

Build the model
---------------

To Build the model, first we need to Create a document-term matrix. Second we need to create a container. At last we fill the container with the machine learning algorithm.

\*\* 1. Create a document-term matrix \*\*

``` r
dtm_mix <- DocumentTermMatrix(mix.corpus)
dtm_mix
```

    ## <<DocumentTermMatrix (documents: 3301, terms: 103178)>>
    ## Non-/sparse entries: 598799/339991779
    ## Sparsity           : 100%
    ## Maximal term length: 196626
    ## Weighting          : term frequency (tf)

``` r
dtm_mix <- removeSparseTerms(dtm_mix, 1-(10/length(mix.corpus)))
dtm_mix
```

    ## <<DocumentTermMatrix (documents: 3301, terms: 5748)>>
    ## Non-/sparse entries: 436889/18537259
    ## Sparsity           : 98%
    ## Maximal term length: 97
    ## Weighting          : term frequency (tf)

\*\* 2 .Create a container\*\*

``` r
# extract meta tag "type" 
classify_labels <- as.vector(unlist(meta(mix.corpus, "type")))
classify_labels <- as.data.frame(classify_labels)

# set up model container; 50/50 split between train and test data
N <- length(classify_labels[,1])
container <- create_container(
  dtm_mix, 
  labels = classify_labels[,1],
  trainSize = 1: round(0.5 * N),
  testSize = (round(0.5 * N)+1) : N,
  virgin = FALSE)

# view the slot of the container
slotNames(container)
```

    ## [1] "training_matrix"       "classification_matrix" "training_codes"       
    ## [4] "testing_codes"         "column_names"          "virgin"

\*\* 3. Creat model by filling the container with the machine learning algorithm.\*\*

``` r
svm_model <- train_model(container, "SVM")
tree_model <- train_model(container,"TREE")
maxent_model <- train_model(container, "MAXENT")
```

\*\* Estimation and Evaluation\*\*

**1. Model output**

``` r
svm_out <- classify_model(container, svm_model)
tree_out <- classify_model(container, tree_model)
maxent_out <- classify_model(container, maxent_model)

labels_out <- data.frame(
correct_label = classify_labels[(round(0.5 * N)+1) : N,1],
svm = as.character(svm_out[,1]),
tree = as.character(tree_out[,1]),
maxent = as.character(maxent_out[,1]),
stringsAsFactors = F)
kable(head(labels_out,10))
```

| correct\_label | svm  | tree | maxent |
|:---------------|:-----|:-----|:-------|
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |
| Spam           | Spam | Spam | Spam   |
| Ham            | Ham  | Ham  | Ham    |
| Ham            | Ham  | Ham  | Ham    |

**2. Model Performance**

The accuracy of each models was evaluated.

``` r
# SVM performance**
svm_table <- table(labels_out[,1] == labels_out[,2])
svm_prop <- round(prop.table(svm_table), 4)

# Random forest performance **
tree_table <- table(labels_out[,1] == labels_out[,3])
tree_prop <- round(prop.table(tree_table), 4)

# Maximum entropy performance**
maxent_table <- table(labels_out[,1] == labels_out[,4])
maxent_prop <- round(prop.table(maxent_table),4)

performance <- rbind(svm_prop,tree_prop ,maxent_prop)
kable(performance)
```

|              |   FALSE|    TRUE|
|--------------|-------:|-------:|
| svm\_prop    |  0.0073|  0.9927|
| tree\_prop   |  0.0073|  0.9927|
| maxent\_prop |  0.0109|  0.9891|

Evaluate the true-positive(spam was predicted as spam), false-positive(ham was predicted as spam), true-negative(ham was predicted as ham), false-negative(spam was predicted as ham).

``` r
svm <- table(correct = labels_out[,1], estimated = labels_out[,2])
svm.df <- as.data.frame(svm)
svm.df$class <- c("t-n","f-n","f-p","t-p")
kable(svm.df)
```

| correct | estimated |  Freq| class |
|:--------|:----------|-----:|:------|
| Ham     | Ham       |  1402| t-n   |
| Spam    | Ham       |     6| f-n   |
| Ham     | Spam      |     6| f-p   |
| Spam    | Spam      |   237| t-p   |

``` r
tree <- table(correct = labels_out[,1], estimated = labels_out[,3])
tree.df <- as.data.frame(tree)
tree.df$class <- c("t-n","f-n","f-p","t-p")

maxent <- table(correct = labels_out[,1], estimated = labels_out[,4])
maxent.df <- as.data.frame(maxent)
maxent.df$class <- c("t-n","f-n","f-p","t-p")

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
svm.df <- cbind(svm.df,svm.df$Freq)
svm.df[,3] <- NULL

accurary <- cbind(svm.df, tree.df$Freq, maxent.df$Freq)
colnames(accurary) <- c("correct","estimated","class","svm.Freq","tree.Freq","maxent.Freq")
accurary$svm.rate <- accurary$`svm.Freq`/sum(accurary$`svm.Freq`)
accurary$tree.rate <- accurary$`tree.Freq`/sum(accurary$`tree.Freq`)
accurary$maxent.rate <- accurary$`maxent.Freq`/sum(accurary$`maxent.Freq`)
kable(accurary)
```

| correct | estimated | class |  svm.Freq|  tree.Freq|  maxent.Freq|   svm.rate|  tree.rate|  maxent.rate|
|:--------|:----------|:------|---------:|----------:|------------:|----------:|----------:|------------:|
| Ham     | Ham       | t-n   |      1402|       1396|         1396|  0.8491823|  0.8455482|    0.8455482|
| Spam    | Ham       | f-n   |         6|          0|            6|  0.0036342|  0.0000000|    0.0036342|
| Ham     | Spam      | f-p   |         6|         12|           12|  0.0036342|  0.0072683|    0.0072683|
| Spam    | Spam      | t-p   |       237|        243|          237|  0.1435494|  0.1471835|    0.1435494|

``` r
par(mfrow=c(1,3))
mosaicplot (svm)
mosaicplot (tree)
mosaicplot (maxent)
```

![](DA607_Project4_files/figure-markdown_github/unnamed-chunk-23-1.png) Overall error rate is very low, 0 - 0.7%. Relatively, Forest Tree method has the lowest false negative and highest false positive.

``` r
accurary_long <- gather(accurary, method, rate, 7:9)

ggplot(data = accurary_long, mapping = aes(x = method, y = rate)) + 
  geom_point(mapping = aes(color = class, size = class)) +
  ggtitle('Accuracy')
```

    ## Warning: Using size for a discrete variable is not advised.

![](DA607_Project4_files/figure-markdown_github/unnamed-chunk-24-1.png)

Majority of the email, 85%, is trully non-spam email and true spam eamils account for around 14.5% of all emails.

Coclusion
---------

In this document classification project I used 50% of emails as my training data and the rest as test data, I used three algorisms - Support vector machines, Forest tree, and Maximum entropy to do the classification. In the test, the accurary of these three models are: gave us over 99.3%, 99.3% and 98.9%. All three methods performed equally well.

Unsupervised text classification
--------------------------------

\*\* LDA: the Latent Dirichlet Allocation\*\*

``` r
library(topicmodels)
```

    ## Warning: package 'topicmodels' was built under R version 3.3.3

``` r
lda_out <- LDA(dtm_mix, 2)
posterior_lda <- posterior(lda_out)
lda_topics <- data.frame(t(posterior_lda$topics))

mean_topic_matrix <- matrix(
NA,
nrow = 2,
ncol = 2,
dimnames = list(names(table(classify_labels)),c("Ham", "Spam"))
)

## Filling matrix
for(i in 1:2){
mean_topic_matrix[i,] <- apply(lda_topics[, which(classify_labels ==
rownames(mean_topic_matrix)[i])], 1, mean)
}

## Outputting rounded matrix
round(mean_topic_matrix, 2)
```

    ##       Ham Spam
    ## Ham  0.92 0.08
    ## Spam 0.77 0.23

``` r
terms(lda_out, 20)
```

    ##       Topic 1             Topic 2                 
    ##  [1,] "sep"               "width"                 
    ##  [2,] "esmtp"             "height"                
    ##  [3,] "localhost"         "widthd"                
    ##  [4,] "aug"               "border"                
    ##  [5,] "receiv"            "tabl"                  
    ##  [6,] "oct"               "helvetica"             
    ##  [7,] "postfix"           "cellpad"               
    ##  [8,] "istreceiv"         "cellspac"              
    ##  [9,] "thu"               "size"                  
    ## [10,] "mon"               "font"                  
    ## [11,] "wed"               "faceari"               
    ## [12,] "from"              "heightd"               
    ## [13,] "jalapeno"          "img"                   
    ## [14,] "jmlocalhost"       "srchttpwwwcnetcombgif" 
    ## [15,] "tue"               "borderd"               
    ## [16,] "dogmaslashnullorg" "heighttdtd"            
    ## [17,] "the"               "srchttphomecnetcombgif"
    ## [18,] "returnpath"        "alt"                   
    ## [19,] "use"               "arial"                 
    ## [20,] "fri"               "srchttpwwwzdnetcombgif"

The terms, at least the first 20, assicated to either ham or spam could not help people to figure out the label. Maybe for unsupervised method it will be more appropriate to only extrac the email body for text analysis.
