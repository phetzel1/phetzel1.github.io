---
layout: post
title:  "Understanding Federal Election Data"
date:   2015-04-27 16:08:23
categories: projects
comments: True
---

##Objective
I've always been fascinated (and frustrated) with the state of campaign finance in the United States of America.  It's because of this fascination that I've turned my attention to the Federal Election Commissions Open Government policy to view the "trackable" donations that candidates are receiving.  I say trackable donations because a great deal of money does not have to be disclosed in an easy to track fashion.

The general purpose of this project is to answer one question:

* How do Political Action Committees spend and receive money over time.

##Data Description

Here are some descriptions of the tables we will download.  They are set up to be relational.


| Name                        | Description                                                                                                                                                                                                                                                                                                                                                          | Data frame |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| Commitees                   | The committee master file contains one record for each committee registered with the Federal Election Commission. This includes federal political action committees and party committees, campaign committees for presidential, house and senate candidates, as well as groups or organizations who are spending money for or against candidates for federal office. | committee  |
| Candidates                  | The candidate master file contains one record for each candidate who has either registered with the Federal Election Commission or appeared on a ballot list prepared by a state elections office.                                                                                                                                                                   | candidates |
| Linkages                    | This file contains one record for each candidate to committee linakge.                                                                                                                                                                                                                                                                                               | linkage    |
| Itemized Records            | The itemized records (miscellaneous transactions) file contains all transactions (contributions, transfers, etc. among federal committees).                                                                                                                                                                                                                          | cm2cm      |
| Contributions to Candidates | The itemized committee contributions file contains each contribution or independent expenditure made by a PAC, party committee, candidate committee, or other federal committee to a candidate during the two-year election cycle.                                                                                                                                   | cm2cand    |
| Individual Contributions    | The individual contributions file contains each contribution from an individual to a federal committee if the contribution was at least $200.                                                                                                                                                                                                                        | indiv2cm   |
##Let's Get Coding

###Set Up

I want to set myself up for success, so I immediately load up some of my favorite packages for data munging.  I also set my working directory and clear my memory.


```r
library(dplyr)
library(ggplot2)
library(reshape2)
setwd("~/R Working Directory/Election_Data")
rm(list=ls())
```

In order to find out the answers to these questions, we can download the data from the [Federal Election Commission](http://www.fec.gov/finance/disclosure/ftpdet.shtml#a2015_2016).  I want to write as little code as possible, so I store the specific links to loop through later.  I am downloading two files for each FEC table: a header file (in .csv format) and the actual data (in a zip file).


```r
#Create a vector with the links leading to CSV files that hold the column names
nameLinks <- c("http://www.fec.gov/finance/disclosure/metadata/cn_header_file.csv", 
              "http://www.fec.gov/finance/disclosure/metadata/indiv_header_file.csv", 
              "http://www.fec.gov/finance/disclosure/metadata/pas2_header_file.csv", 
              "http://www.fec.gov/finance/disclosure/metadata/cm_header_file.csv", 
              "http://www.fec.gov/finance/disclosure/metadata/ccl_header_file.csv",
              "http://www.fec.gov/finance/disclosure/metadata/oth_header_file.csv")

#Create a vector with the links leading to the 6 data frames
datLinks <- c("ftp://ftp.fec.gov/FEC/2016/cn16.zip",
              "ftp://ftp.fec.gov/FEC/2016/indiv16.zip",
              "ftp://ftp.fec.gov/FEC/2016/pas216.zip",
              "ftp://ftp.fec.gov/FEC/2016/cm16.zip",
              "ftp://ftp.fec.gov/FEC/2016/ccl16.zip",
              "ftp://ftp.fec.gov/FEC/2016/oth16.zip")

#Create a vector with the file names of each of the tables
fileName <- c('cn.txt',
              'itcont.txt',
              'itpas2.txt',
              'cm.txt',
              'ccl.txt',
              'itoth.txt')
```

FEC's site gives the columns and associated data types, so I create vectors with the those types to pass into the loop.  This will ensure that nothing gets misread.  I also create a list of 6 empty data frames to import the data into.


```r
#Create vectors describing the class of each column in each table
candClasses <- rep('character', 15)
indColClasses <- c(rep("character", 14), "numeric", rep("character",6))
comContrClasses <- c(rep('character', 14), 'numeric', rep('character', 7))
comClasses <- rep('character', 15)
linkageClasses <- rep('character', 7)
cm2cmClasses <- c(rep('character', 14), 'numeric', rep('character', 6))

#Place these vectors in a list in preparation for a loop
iterClasses <- list(candClasses, indColClasses, comContrClasses, comClasses, linkageClasses, cm2cmClasses)

#Create a list of 6 blank data frames in preparation for a loop
frames <- list(rep(data.frame(),6))
```


###Getting the Data

Now it's time for the loop.  The basic structure of the loop is as follows

1. Initialize a temporary file
2. Download the header file from the nameLinks vector and place into the first temp file
3. Initialize a second temp file
4. Download the data table referenced by the fileName vector and use the iterClasses vector to define column types.  Unzip and place the table into the list of data frames
5. Define the names of the new data frame as the data in the tempNames file
6. Detach the temp files


```r
for (i in 1:length(nameLinks)){
    tempNames <- tempfile()
    download.file(nameLinks[i], tempNames, method = "curl")
    temp <- tempfile()
    download.file(datLinks[i], temp, method = "curl")
    frames[[i]] <- read.table(unz(temp,fileName[i]),header = F, sep = "|",comment.char = "",stringsAsFactors = F,
                       colClasses = unlist(iterClasses[i]), quote = "")
    
    names(frames[[i]]) <- read.csv(tempNames,header = F, stringsAsFactor = F)
    unlink(temp)
    unlink(tempNames)
    remove(temp,tempNames)
}
```

By the time this is over, you should have a list with 6 full data frames.  The next step is to extract the dataframes into variables we can remember.  After the data frames have been moved into separate variables, we can clean up our memory.


```r
#Place the items in the frames list into specific variables
#Delete the list items as they are used to save memory
candidates <- as.data.frame(frames[[1]])
frames[[1]] <- NULL
indiv2cm <- as.data.frame(frames[[1]])
frames[[1]] <- NULL
cm2cand <- as.data.frame(frames[[1]])
frames[[1]] <- NULL
committee <- as.data.frame(frames[[1]])
frames[[1]] <- NULL
linkage <- as.data.frame(frames[[1]])
frames[[1]] <- NULL
cm2cm <- as.data.frame(frames[[1]])

#Remove variables not being used
remove(frames, candClasses, cm2cmClasses, comClasses, comContrClasses, datLinks, fileName, 
       i, indColClasses, iterClasses, linkageClasses, nameLinks)
```

###Transforming the Data

Since we'll be doing some time series analysis, we can convert any data fields into the proper format.


```r
#Change to date type where applicable
indiv2cm$TRANSACTION_DT <- as.Date(indiv2cm$TRANSACTION_DT,format = "%m%d%Y")
cm2cand$TRANSACTION_DT <- as.Date(cm2cand$TRANSACTION_DT, format = '%m%d%Y')
cm2cm$TRANSACTION_DT <- as.Date(cm2cm$TRANSACTION_DT, format = '%m%d%Y')
```

This is where it gets a little tricky.  My first goal is to see how much money a PAC has spent or received on any given day.  That means I need to have a row for every day per every PAC.  The code below makes a new data frame (cmSmall), which is the committee data frame with less variables.  We then create a vector, days, which will have a day for every day so far this year.  We then create a data frame with every day this year, for every PAC.  We can then join the cmSmall data frame to add in relevent variables


```r
cmSmall <- select(committee, CMTE_ID, CMTE_NM, CMTE_PTY_AFFILIATION,ORG_TP)

days <- seq(as.Date('2015-01-01'), as.Date(Sys.Date()), 'day')
cmGrpMoney <- arrange(expand.grid(CMTE_ID = cmSmall$CMTE_ID, OnDate = days), CMTE_ID)

cmGrpMoney <- cmGrpMoney %>%
    left_join(cmSmall, by = ('CMTE_ID' = 'CMTE_ID'))
```

In our data, there are cases where PACs are receiving (or spending) money through multiple transactions a day.  We need to aggregate that data by PAC by day:


```r
#Shows money spent from individuals on PACs
indivAssets <- indiv2cm %>%
    group_by(CMTE_ID, TRANSACTION_DT) %>%
    arrange(TRANSACTION_DT) %>%
    summarise(IndSum = sum(TRANSACTION_AMT))

#Shows how much a committee has spent on other committees
cm2cmLiabilities <- cm2cm %>%
    group_by(CMTE_ID, TRANSACTION_DT) %>%
    arrange(TRANSACTION_DT) %>%
    summarise(Liabilities = sum(TRANSACTION_AMT)) %>%
    select(CMTE_ID,TRANSACTION_DT, Liabilities)

#Shows how much a commmittee has received from other committees
cm2cmAssets <- cm2cm %>%
    group_by(OTHER_ID, TRANSACTION_DT) %>%
    arrange(TRANSACTION_DT) %>%
    summarise(Assets = sum(TRANSACTION_AMT)) %>%
    select(OTHER_ID,TRANSACTION_DT, Assets)

#Shows how much a committee has spent on candidates
cm2candLiabilities <- cm2cand %>%
    group_by(CMTE_ID, TRANSACTION_DT) %>%
    arrange(TRANSACTION_DT) %>%
    summarise(ToCand= sum(TRANSACTION_AMT)) %>%
    select(CMTE_ID,TRANSACTION_DT, ToCand)

#Shows how much a candidate committee has received from other committees (should be 0)
cm2candAssets <- cm2cand %>%
    group_by(OTHER_ID, TRANSACTION_DT) %>%
    arrange(TRANSACTION_DT) %>%
    summarise(CandReceived = sum(TRANSACTION_AMT)) %>%
    select(OTHER_ID,TRANSACTION_DT, CandReceived)
```


###Bringing It All Together

It's time to combine all of these assets and liabilities into one table.  Hadley Wickham's dplyr (also used above) makes this process simple.  We also want to make all NA values zero (that came as a result of the left join).


```r
#Join the Asset and Liability data frames to the master group
cmGrpMoney <- cmGrpMoney %>%
    left_join(indivAssets, by = c('CMTE_ID' = 'CMTE_ID', 'OnDate' = 'TRANSACTION_DT')) %>%
    left_join(cm2cmAssets, by = c('CMTE_ID' = 'OTHER_ID', 'OnDate' = 'TRANSACTION_DT')) %>%
    left_join(cm2cmLiabilities, by = c('CMTE_ID' = 'CMTE_ID', 'OnDate' = 'TRANSACTION_DT')) %>%
    left_join(cm2candLiabilities, by = c('CMTE_ID' = 'CMTE_ID', 'OnDate' = 'TRANSACTION_DT')) %>%
    left_join(cm2candAssets, by = c('CMTE_ID' = 'OTHER_ID', 'OnDate' = 'TRANSACTION_DT'))


#Replace NAs induced by the join to 0
cmGrpMoney[is.na(cmGrpMoney)] <- 0
```

We want to view total assets and total liabilities over time.  This is why were are using the cumulative sum function, then adding together the asset columns and liability columns, separately.


```r
cmGrpMoney <- cmGrpMoney %>%
    group_by(CMTE_ID) %>%
    mutate(FromCommittees = cumsum(Assets), ReceivedCandidates = cumsum(CandReceived),
           Funds = cumsum(IndSum), SpentCommittees = cumsum(Liabilities), SpentCandidates = cumsum(ToCand), 
           TotalRaised = FromCommittees+Funds+ReceivedCandidates, TotalSpent = SpentCommittees+SpentCandidates) %>%
    select(CMTE_ID, CMTE_NM, OnDate, Funds, FromCommittees, TotalRaised, SpentCommittees, SpentCandidates, TotalSpent)
```

We have a massive dataset right now but how much of it is useful?  In the beginning of the exercise, we made a data frame using all of the listed PACs.  Most of these PACs have not given nor received money yet this year, so we want to filter them out.


```r
check <- cmGrpMoney %>%
    group_by(CMTE_ID) %>%
    summarise(raised = sum(TotalRaised), spent = sum(TotalSpent)) %>%
    filter(raised == 0 & spent == 0) %>%
    select(CMTE_ID)

check <- check$CMTE_ID

cmGrpMoney <- cmGrpMoney[(!cmGrpMoney$CMTE_ID %in% check),]
```

##Conclusion

The goal of this work was simply to get the data into the correct format for analysis.  We can plug this new data output into a Shiny application to more dynamically view Assets and Liabilities of PACs over time.  Please go to my [Shiny Application](https://phetzel1.shinyapps.io/Election_Data/) and play around.  Start typing in a company name and see how active their PAC is!

##Shiny Code


```r
library(shiny)
require(rCharts)

#ui.R
shinyUI(fluidPage(
    titlePanel('PAC Fundraising vs. Spending'),
    sidebarPanel(
        selectizeInput('committee', choices = NULL, label = 'Search for a Committee')
    ),
    mainPanel(
        showOutput("raised", 'morris')
        )
    )
)

#server.R
library(shiny)
library(rCharts)
library(data.table)

cmGrpMoney <- fread("cmGrpMoney.txt", sep = '|')
setnames(cmGrpMoney, c('CMTE_ID', 'CMTE_NM', 'OnDate', 'Funds', 'FromCommittees', 'TotalRaised', 'SpentCommittees', 'SpentCandidates', 'TotalSpent'))
orgs <- fread('orgs.txt', sep = '\n', header = F)$V1

shinyServer(function(input, output, session){
    
    updateSelectizeInput(session, 'committee', choices = orgs, server = T, options = list(placeholder = 'Start typing!'))
    
    output$raised <- renderChart({
        dat <- cmGrpMoney[cmGrpMoney$CMTE_NM == input$committee,]
        p1 <- mPlot(x = 'OnDate', y = list('TotalRaised', 'TotalSpent'), data = dat, type = 'Line', 
                    labels = c('Total Raised', 'Total Spent'), 
                    pointSize = 0, lineSize = 1)

        p1$set(dom = 'raised', lineColors = c('green', 'red'))
        return(p1)})


})
```


