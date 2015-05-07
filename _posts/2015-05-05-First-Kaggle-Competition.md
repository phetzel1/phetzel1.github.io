---
layout: post
title:  "First Kaggle Competition"
date:   2015-05-05 00:00:00
categories: posts
comments: True
---

##Background

I'm currently taking an excellent course on [edX.org](https://www.edx.org/) called "The Analytics Edge," taught by MIT.  The course structure helps the student learn a different statistical model every week.  For the past three weeks, the course sponsored a private [Kaggle](https://www.kaggle.com/) competition.  In this blog post, I will walk you through the process that I used to place in the top 10% of the competition.

##Purpose

The competition sponsor chose to study and analyze the popularity of New York Times website articles.  The competitors were to create a model that would accurately predict whether or not an article would be popular, with popularity being defined as 25 or more comments on the article.  Let's grab the data and look how it is structured.




```r
train1 <- read.csv('NYTimesBlogTrain.csv', stringsAsFactors = F)
test1 <- read.csv('NYTimesBlogTest.csv', stringsAsFactors = F)
str(train1)
```

```
## 'data.frame':	6532 obs. of  10 variables:
##  $ NewsDesk      : chr  "Business" "Culture" "Business" "Business" ...
##  $ SectionName   : chr  "Crosswords/Games" "Arts" "Business Day" "Business Day" ...
##  $ SubsectionName: chr  "" "" "Dealbook" "Dealbook" ...
##  $ Headline      : chr  "More School Daze" "New 96-Page Murakami Work Coming in December" "Public Pension Funds Stay Mum on Corporate Expats" "Boot Camp for Bankers" ...
##  $ Snippet       : chr  "A puzzle from Ethan Cooper that reminds me that a bill is due." "The Strange Library will arrive just three and a half months after Mr. Murakamis latest novel, Colorless Tsukuru Tazaki and His"| __truncated__ "Public pension funds have major stakes in American companies moving overseas to cut their tax bills. But they are saying little"| __truncated__ "As they struggle to find new business to bolster sluggish earnings, banks consider the nations 25 million veterans and service "| __truncated__ ...
##  $ Abstract      : chr  "A puzzle from Ethan Cooper that reminds me that a bill is due." "The Strange Library will arrive just three and a half months after Mr. Murakamis latest novel, Colorless Tsukuru Tazaki and His"| __truncated__ "Public pension funds have major stakes in American companies moving overseas to cut their tax bills. But they are saying little"| __truncated__ "As they struggle to find new business to bolster sluggish earnings, banks consider the nations 25 million veterans and service "| __truncated__ ...
##  $ WordCount     : int  508 285 1211 1405 181 245 258 893 1077 188 ...
##  $ PubDate       : chr  "2014-09-01 22:00:09" "2014-09-01 21:14:07" "2014-09-01 21:05:36" "2014-09-01 20:43:34" ...
##  $ Popular       : int  1 0 0 1 1 1 0 1 1 0 ...
##  $ UniqueID      : int  1 2 3 4 5 6 7 8 9 10 ...
```

Now let's look at the first couple of rows (the `kable` function is just a way to make the chart prettier in HTML):


```r
kable(head(train1))
```



|NewsDesk   |SectionName        |SubsectionName   |Headline                                            |Snippet                                                                                                                                                                                                                           |Abstract                                                                                                                                                                                                                           |WordCount  |PubDate                |Popular   |UniqueID|
|---------  |-----------------  |---------------  |-------------------------------------------------  |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  |:----------  |:--------------------:  |:--------:  |:---------:|
|Business   |Crosswords/Games   |                |More School Daze                                    |A puzzle from Ethan Cooper that reminds me that a bill is due.                                                                                                                                                                    |A puzzle from Ethan Cooper that reminds me that a bill is due.                                                                                                                                                                     |      508  |2014-09-01 22:00:09    |      1   |       1|
|Culture    |Arts               |                 |New 96-Page Murakami Work Coming in December       |The Strange Library will arrive just three and a half months after Mr. Murakamis latest novel, Colorless Tsukuru Tazaki and His Years of Pilgrimage.                                                                              |The Strange Library will arrive just three and a half months after Mr. Murakamis latest novel, Colorless Tsukuru Tazaki and His Years of Pilgrimage.                                                                                |     285  |2014-09-01 21:14:07     |     0   |       2|
|Business   |Business Day       |Dealbook         |Public Pension Funds Stay Mum on Corporate Expats   |Public pension funds have major stakes in American companies moving overseas to cut their tax bills. But they are saying little about the strategy, which could hurt the nations tax base.                                        |Public pension funds have major stakes in American companies moving overseas to cut their tax bills. But they are saying little about the strategy, which could hurt the nations tax base.                                           |   1211  |2014-09-01 21:05:36     |     0   |       3|
|Business   |Business Day       |Dealbook         |Boot Camp for Bankers                               |As they struggle to find new business to bolster sluggish earnings, banks consider the nations 25 million veterans and service members ideal customers.                                                                           |As they struggle to find new business to bolster sluggish earnings, banks consider the nations 25 million veterans and service members ideal customers.                                                                              |   1405  |2014-09-01 20:43:34    |      1  |        4|
|Science    |Health             |                 |Of Little Help to Older Knees                       |Middle-aged and older patients are unlikely to benefit in the long term from surgery to repair tears in the meniscus, pads of cartilage in the knee, a new review of studies has found.                                           |Middle-aged and older patients are unlikely to benefit in the long term from surgery to repair tears in the meniscus, pads of cartilage in the knee, a new review of studies has found.                                               |   181  |2014-09-01 18:58:51    |      1  |        5|
|Science    |Health             |                 |A Benefit of Legal Marijuana                        |A new study has found evidence that legal access to marijuana is associated with fewer opioid overdose deaths, but researchers said their findings should not be used as the basis for the wide adoption of legalized cannabis.   |A new study has found evidence that legal access to marijuana is associated with fewer opioid overdose deaths, but researchers said their findings should not be used as the basis for the wide adoption of legalized cannabis.        |  245  |2014-09-01 18:52:22    |      1  |        6|

##What's the Plan?

Right away I had a general direction of how I wanted to make my model.  Here are some things I decided immediately:

1. This is a binary classification problem.  In other words, we are going to try to predict that an article is either popular, or not popular.  

2. I do not want to use a great deal of text analytics.  Some of my peers decided to use "Bag of Words" for feature extraction.  More on this later.

3. The data allows for time-based feature extraction (i.e. I can pull out the hour and day of an article).

4. There are many recurring series of articles, many of which are either very popular or not at all popular.

With these general observations in mind, I'm going to start adding features to this data-set to later put into a model.

##Adding Features (and General Maintenance)

Since some of the features we will eventually put into our model are either categorical or binary, we will preprocess the training and testing data-set together.  This ensures that the training and testing set will have the same set of factors when we eventually split them apart again.  For the `complete` data frame, we will not include the `Popular` variable.


```r
complete <- rbind(train1[,c(1:8,10)], test1)
```

The weekday and time an article was published may be useful in determining its popularity.  I'm going to separate the publish date feature into discrete `Weekday` and `Hour` features.  They will be stored as factors.


```r
complete$PubDate = strptime(complete$PubDate, "%Y-%m-%d %H:%M:%S")
complete$Weekday <- as.factor(complete$PubDate$wday)
complete$Hour <- as.factor(complete$PubDate$hour)
```

I have also noticed that the `NewsDesk`, `SectionName`, and `SubSectionName` are not consistently filled out.  Therefore, I want to collapse these fields into one feature.  My hope is that I will be able to better group the data with a more complete set of section categories.  I use a host of `ifelse` functions to fill in my new `Section` feature.


```r
complete$Section <- ifelse(complete$NewsDesk == 'TStyle', 'Style', '')
complete$Section <- ifelse(complete$NewsDesk == 'Styles' & complete$Section == '', 'Style', complete$Section)
complete$Section <- ifelse(complete$NewsDesk == 'Foreign' & complete$Section == '', 'World', complete$Section)
complete$Section <- ifelse(complete$SubsectionName == 'Education' & complete$Section == '', 'Education', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'National' | complete$SectionName == 'U.S.') & complete$Section == '', 'Politics', complete$Section)
complete$Section <- ifelse(complete$NewsDesk == 'Science' | complete$SectionName == 'Health' & complete$Section == '', 'Health', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'OpEd' | complete$SectionName == 'Opinion') & complete$Section == '', 'Opinion', complete$Section)
complete$Section <- ifelse(complete$SectionName == 'Crosswords/Games' & complete$Section == '', complete$SectionName, complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Culture' | complete$SectionName == 'Arts') & complete$Section == '', 'Culture', complete$Section)
complete$Section <- ifelse((complete$SectionName == 'Technology' | complete$SectionName == 'Open') & complete$Section == '', 'Technology', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Business' | complete$SectionName == 'Business Day') & complete$Section == '', 'Business', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Travel' | complete$SectionName == 'Travel') & complete$Section == '', 'Travel', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Sports' | complete$SectionName == 'Sports') & complete$Section == '', 'Sports', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Magazine' | complete$SectionName == 'Magazine') & complete$Section == '', 'Magazine', complete$Section)
complete$Section <- ifelse(complete$SectionName == 'Multimedia' & complete$Section == '', 'Multimedia', complete$Section)
complete$Section <- ifelse((complete$NewsDesk == 'Metro' | complete$SectionName == 'N.Y./Region' | complete$SectionName == 'N.Y. / Region') & complete$Section == '', 'Metro', complete$Section )
```

After doing some digging around the training set, I notice that headlines that ask questions seem to have a higher rate of popularity.  I introduce the `Question` variable to capture this.  The `grepl` function returns a `TRUE` or `FALSE` value if any value in the regular expression is in the headline.  For more on regular expressions, I would suggest [this link](https://msdn.microsoft.com/en-us/library/az24scfc%28v=vs.110%29.aspx).


```r
complete$Question <- as.numeric(grepl('\\?|^How|^Should|^When|^Why|^What', complete$Headline, perl = T))
```

I also want to add two new features in reference to the fourth point in the "What's the Plan" section above.  I look through the data set and note which recurring articles are popular, and which are not.


```r
pop_headlines <- c('^Readers Respond', '^No Comment Necessary', '^Living With Cancer', '^Ask Well', '^Facts', 'Quandary:', '^Think Like a Doctor')
unpop_headlines <- c('^[0-9]', "^Analytics:", "^Behind the Cover Story", "^Big Ticket", "^Book Review Podcast",
                     "^Classical Playlist", "^CMJ 2014", "^Daily Clip Report", "^Daily Report", "^Election 2014", 
                     "Fashion Week", "^First Draft Focus", "^First Draft Video", "^Friday Night Music", "^From The Upshot",
                     "^In Performance", "^International Arts Events", "^Joe on WNYC", "^Lunchtime Laughs", 
                     "^Morning Agenda", "^New York City's Week in Pictures", "^New York Parking Alert", "Pictures of the Day", 
                     "^NYTLNreads", "^On This Day", "^Poetry Pairings", "^Politics Helpline", "^Popcast", "^Q\\. and A\\.", 
                     "^Reading the Times", "^Student Crossword", "^Sunday Breakfast Menu", "^Test Yourself", "^Text to Text", 
                     "^The Daily Gift", "^Throwback Thursday", "^Today in Politics", "^Today in Small Business", 
                     "^Toronto Festival 2014", "^Tune Into The Times", "^Tune In to The Times", "^Under Cover", "^Variety", 
                     "^Verbatim", "^Video Reviews", "^Walkabout", "^Weekend Reading", "^Weekly News Quiz", "Weekly Wrap", 
                     "^What We're", "^What's Going On", "^Word of the Day")

complete$Section <- as.factor(complete$Section)
```

This allows me to plug these vectors into a `grepl` command.  The `collapse` parameter in the `paste` command places a pipe character between each of the vector items.  The pipe character tells R to return `TRUE` if any of the vector items are present in the headline.  The new features are called `Bonus` and `NotBonus`, which are simple indicator variables.  An observation will receive a 1 in the `Bonus` variable if they are present in the `pop_headlines` vector.  An observation will also receive a 1 in the `NotBonus` variable if they are _not_ present in the `unpop_headlines`.  


```r
complete$Bonus <- as.factor(ifelse(
    grepl(paste(pop_headlines, collapse = '|'), fixed = F, complete$Headline), 1, 0
))

complete$NotBonus <- as.factor(ifelse(
    grepl(paste(unpop_headlines, collapse = '|') , fixed = F, complete$Headline) | complete$SubsectionName == 'Asia Pacific', 0, 1
))
```

I also want to reduce the `WordCount` to a logarithmic scale (I'm adding 1 to the `WordCount` variable to protect against observations with a `WordCount` of 0).


```r
complete$WordCount <- log(1 + complete$WordCount)
```

That's all the feature extraction we'll do for this data-set; we don't want to over-fit.  The last step is to split the data back into training and testing sets, then adding back the response variable.


```r
train <- head(complete, nrow(train1))
test <- tail(complete, nrow(test1))

train$Popular <- train1$Popular
```


##Forming a Model

I'm going to form three models:

1. Logistic Regression

2. Random Forest

3. Gradient Boosted Tree

###Logistic Regression

A logistic regression may be a decent model choice because it lends itself to binary classification problems.  We will use the `glm` command, and use the `Section`, `Weekday`, `Question`, `Bonus`, and `NotBonus` to predict `Popular`.  


```r
model_glm <- glm(Popular ~ Section + Weekday + Question + Bonus + NotBonus, family = binomial, data = train)
```

Let's see how this model performed on the training set.  The `response` parameter notifies R to return probability rather than the predicted class.  For this model, we'll measure the accuracy (True Negatives + True Positives / # of observations)


```r
glm_pred <- predict(model_glm, type = 'response')

t <- table(train$Popular, glm_pred > .5)
(t[1, 1] + t[2, 2])/nrow(train)
```

```
## [1] 0.8948255
```

That accuracy is okay, but we can do better by using more complex statistical models.

###Random Forest

The random forest model is a strong choice for this problem because it will be able to capture non-linear relationships.  We will use the `train` function from the `caret` package in order to cross validate the random forest model.  The competition is scoring off of the model with the best Area Under the Curve (AUC), which is why we are using `ROC` as our metric.  The `pROC` package allows us to use the `ROC` metric.

The `train` function does not like when the response variable (Popular) is numeric, so the first step is to change the values to either "yes" or "no."  Note that using the train function classification calls for the response variable to be a factor.

Full disclosure: I know from previous iterations that the best `mtry` value is in the range of 8 - 15.  Additionally in my original model, I used 10 fold cross validation with `ntree` set at 1000.  This took a loooooooooooong time on my puny laptop, so I've purposely made the model less computationally expensive for the purpose of writing this post.


```r
require(randomForest)
require(caret)
require(e1071)
require(pROC)

train$Popular <- as.factor(train1$Popular)
train$Popular <- as.factor(ifelse(train$Popular == "0", 'no', 'yes'))

rfGrid = expand.grid(.mtry = 8:15)

set.seed(1)
model_rf_cv <- train(Popular ~ Question + Bonus + NotBonus + Section + Weekday + Hour, data = train, metric = 'ROC', method = 'rf', 
                  ntree = 1000, nodesize = 5, tuneGrid = rfGrid, 
                  trControl = trainControl('cv', number = 5, allowParallel = T, 
                                           classProbs = T, summaryFunction = twoClassSummary))

model_rf_cv
```

```
## Random Forest 
## 
## 6532 samples
##   15 predictor
##    2 classes: 'no', 'yes' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## 
## Summary of sample sizes: 5226, 5226, 5226, 5225, 5225 
## 
## Resampling results across tuning parameters:
## 
##   mtry  ROC        Sens       Spec       ROC SD      Sens SD   
##    8    0.8821224  0.9571629  0.6066231  0.01077888  0.01019918
##    9    0.8828425  0.9555083  0.6148716  0.01506417  0.01247153
##   10    0.8828170  0.9547730  0.6075615  0.01149230  0.01233141
##   11    0.8796088  0.9542215  0.6112103  0.01282544  0.01224512
##   12    0.8807107  0.9529348  0.6130451  0.01158821  0.01289943
##   13    0.8806497  0.9509125  0.6112228  0.01244417  0.01257015
##   14    0.8804412  0.9509125  0.6121277  0.01218954  0.01216024
##   15    0.8816241  0.9507285  0.6038959  0.01424000  0.01246921
##   Spec SD   
##   0.01984507
##   0.02522791
##   0.03101629
##   0.02453448
##   0.02869649
##   0.03256856
##   0.02642122
##   0.02793863
## 
## ROC was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 9.
```

The above model takes a while to run because the function runs is performing k-fold cross validation over five folds, for eight different values of `mtry` (`mtry` is the number of features sampled for splitting at each node).


###Stochastic Gradient Boosting method

This model was new to me.  I cannot go into the specifics of what makes the model work, but from what I understand it is an ensemble method that allows for gradually "learning" the best model.  I've linked to a superb definition of this method [here](http://www.quora.com/What-is-Gradient-Boosting-Models-and-Random-Forests-using-layman-terms#).  

Since I'm unfamiliar with the model, I'm going to be pretty broad with the tuning grid and let `train` pick the best model for me.


```r
gbmGrid <- expand.grid(interaction.depth = (1:3) * 2,
                      n.trees = (1:6)*250, shrinkage = .1, n.minobsinnode = 10)
set.seed(2)
model_cv <- train(Popular ~ Section + WordCount + Weekday + Hour + Question + Bonus + NotBonus, data = train, metric = 'ROC', method = 'gbm', tuneGrid = gbmGrid, 
                  trControl = trainControl('cv', number = 5, allowParallel = T, classProbs = T, summaryFunction = twoClassSummary))
```

```r
model_cv
```

```
## Stochastic Gradient Boosting 
## 
## 6532 samples
##   15 predictor
##    2 classes: 'no', 'yes' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## 
## Summary of sample sizes: 5225, 5226, 5225, 5226, 5226 
## 
## Resampling results across tuning parameters:
## 
##   interaction.depth  n.trees  ROC        Sens       Spec       ROC SD     
##   2                   250     0.9325765  0.9667213  0.6029115  0.007294326
##   2                   500     0.9344560  0.9628597  0.6367685  0.007674865
##   2                   750     0.9342770  0.9612056  0.6495916  0.007449478
##   2                  1000     0.9341366  0.9617574  0.6477357  0.008137925
##   2                  1250     0.9341734  0.9612058  0.6523271  0.007813192
##   2                  1500     0.9332999  0.9589999  0.6541620  0.008806376
##   4                   250     0.9341907  0.9606536  0.6376817  0.006881781
##   4                   500     0.9331252  0.9577130  0.6395208  0.007874766
##   4                   750     0.9317527  0.9555068  0.6459218  0.008506607
##   4                  1000     0.9300016  0.9551391  0.6450170  0.008412948
##   4                  1250     0.9282388  0.9523818  0.6367810  0.008633019
##   4                  1500     0.9266985  0.9523809  0.6303758  0.008231190
##   6                   250     0.9339229  0.9606538  0.6367936  0.006858522
##   6                   500     0.9309758  0.9566094  0.6440953  0.007486497
##   6                   750     0.9286904  0.9533004  0.6431737  0.008177307
##   6                  1000     0.9251800  0.9520138  0.6367601  0.007860551
##   6                  1250     0.9234056  0.9516458  0.6431653  0.007761015
##   6                  1500     0.9207935  0.9503583  0.6386159  0.007671433
##   Sens SD      Spec SD   
##   0.004970930  0.02443406
##   0.006263708  0.02413679
##   0.006010095  0.02179991
##   0.006616881  0.02687704
##   0.006383519  0.01929270
##   0.006353768  0.02208576
##   0.005576795  0.02060370
##   0.008497721  0.02965727
##   0.008105456  0.02100855
##   0.008021189  0.02619897
##   0.006665764  0.02085512
##   0.007012165  0.02202656
##   0.007733225  0.03479841
##   0.007366771  0.03202290
##   0.006734193  0.03120297
##   0.008893539  0.03020820
##   0.007758948  0.03335144
##   0.008793118  0.02107174
## 
## Tuning parameter 'shrinkage' was held constant at a value of 0.1
## 
## Tuning parameter 'n.minobsinnode' was held constant at a value of 10
## ROC was used to select the optimal model using  the largest value.
## The final values used for the model were n.trees = 500,
##  interaction.depth = 2, shrinkage = 0.1 and n.minobsinnode = 10.
```

##Final Submission

The Boosted Tree method had a stronger performance on the training data, however I was worried about over-fitting the model.  In order to protect against this, I took the predictions of both models and averaged them together.


```r
test_gbm_cv_pred <- predict(model_cv, newdata = test, type = 'prob')[, 2]
test_rf_cv_pred <- predict(model_rf_cv, newdata = test, type = 'prob')[, 2]

avg <- as.data.frame(cbind(test_rf_cv_pred, test_gbm_cv_pred))

avg$avg <- (avg$test_rf_cv_pred+avg$test_gbm_cv_pred)/2
MySubmission = data.frame(UniqueID = test1$UniqueID, Probability1 = avg$avg)
```

##Results and Lessons Learned

I ended up scoring a .90277 AUC, which was good enough to sneak into the top 10 percent.  Through this exercise, I learned how important cross validating a model can be in machine learning.  When initially forming models, there is a strong temptation to add in as many variables as possible in order to achieve high _training set prediction_ accuracy.  Don't do this! Simpler models are more likely to succeed with test set predictions.  Cross validating is an excellent way to ensure that you are not over-fitting.

I was tempted to use a "Bag of Words" approach and throw in a bunch of words that showed up in the Abstract as features.  I ended up not using this approach, but rather looked for recurring series of articles, then manually added them as features.

Lastly, I learned about a new model, Boosted Trees, which is another ensemble method that can be used for classification and regression.  If anyone is actually reading this, let me know in the comments if you have any questions, or noticed something that I could have done better!
