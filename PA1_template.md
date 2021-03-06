---
## "Reproducible Research: Peer Assessment 1"
---
title: ""Reproducible Research: Peer Assessment 1""
author: Lex Knape
date: March 15, 2015
output: md_document
---

### **Loading and preprocessing the data**




```r
library("plyr")
library("ggplot2")
library("knitr")
library("rmarkdown")
```

Unzip the activity.zip and read the activity.csv file

```r
unzip("activity.zip")
activity <- read.csv("activity.csv", header=T)
```

## **What is mean total number of steps taken per day?**

Before we can calculate the number of steps per day we must
set the class for Activity$date to be date instead of the current class factor

```r
class(activity$date)
```

```
## [1] "factor"
```

```r
activityDate <-transform(activity, date = as.Date(date))
class(activityDate$date)
```

```
## [1] "Date"
```

```r
# Class of date is a factor needs to be date.
```

We can now calculate the total number of steps per day.

```r
MstepsPerDay <- ddply(activity, ~date, summarise, steps = sum(steps))
```

In order to show the number of steps per day I will show the result of the stepsPerDay in a histogram

```r
p <- ggplot (MstepsPerDay, aes (steps))
p <- p + geom_histogram(fill = "yellow", color = "black") + theme_bw(base_family = "Verdana")
p + ggtitle ("Total number of steps per day")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk Histogram mean](figure/Histogram mean-1.png) 

The plot shows missing values and to calculate the mean we must get ride of these
Get ride of the missing values in DATA MstepsPerDay and calculate number of steps

```r
activityStepsClean <- activity[complete.cases(activity[,1]), ]
stepsPerDayClean <- ddply(activityStepsClean, ~date, summarise, steps = sum(steps))
```

Finally we calculate the mean of the total number os steps per day without NA.

```r
meanstepsPerDay <- mean(MstepsPerDay$steps, na.rm=T)
medianstepsPerDay <- median(MstepsPerDay$steps, na.rm=T)
```

The mean total number of steps taken per day is 10766.

## What is the average daily activity pattern?

As the device collects data at 5 minute intervals through out the day we need a plot (time series) 
of the activity to get a feel. First calculate the steps taken per interval


```r
StepsPerIntervalAverage <- ddply (activity, ~interval, summarise, mean = mean (steps, na.rm=T))
```

I make a time series plot (<code>i.e. type = "l"</code>) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
p <- ggplot (StepsPerIntervalAverage, aes (interval,mean))
p + geom_line(col= "steelblue") + theme_bw(base_family = "Verdana")
```

![plot of chunk Histogram mean tot. steps per day](figure/Histogram mean tot. steps per day-1.png) 

The Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
The 835 5-minute interval contains the maximum number of steps on average across all the days in the dataset


## Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
The dataset activity with all values shows 17.568 observation while activityClean shows 15.264 observations 
which results in 2.304 missing vales

I will test the above via an r script.

```r
RowsOfNAs <- is.na(activity$steps)
summary(RowsOfNAs)
```

```
##    Mode   FALSE    TRUE    NA's 
## logical   15264    2304       0
```
The double check shows the summary of NA's TRUE 2.304. The total number of missing values is 2.304

I choose the strategy to fill the missing values in the dataset with the mean for that 5-minute interval. 

```r
Replace <- function(NAMean) {
        ddply(NAMean, ~interval, function (dd) {
                steps <- dd$steps
                dd$steps[is.na(steps)] <- mean (steps, na.rm =T)
                return(dd)
        })
}
```

Create a new dataset that is equal to the original dataset but with the missing data filled in. 

```r
imputeNA <- Replace(activity)
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

In order to make the histogram of total steps I calculate the total number of steps

```r
impute <- ddply(imputeNA, ~date, summarise, steps = sum(steps))
```

A histogram of the total number of steps taken each day

```r
p <- ggplot(impute, aes(steps))
p <- p + geom_histogram(fill = "steelblue", color = "red")
p <- p + ggtitle (" The total number of steps taken each day")
p + xlab("Steps per day")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk Plot](figure/Plot-1.png) 

The mean and median total number of steps taken per dayare calculated 

```r
meanstepsPerDayClean <- mean(impute$steps)
medianstepsPerDayClean <- median(impute$steps)
```
The mean steps per day is 10.766 and are the same as the none imputeed value
The median is 10.765 and with a difference of 1 it can be neglected

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels ??? weekday and weekend indicating whether a given date is a weekday or weekend day.


```r
weekParts <- c("Weekday", "Weekend")

date2weekpart <- function(date) {
        day <- weekdays(date)
        part <- factor ("Weekday", weekParts)
        if (day %in% c("Saturday", "Sunday"))
                part <-factor("Weekend", weekParts)
        return(part)
}
```

Change the class of the imputeNA$date from factor towards Date


```r
class(imputeNA$date)
```

```
## [1] "factor"
```

```r
imputeNADate <-transform(imputeNA, date = as.Date(date))
class(imputeNADate$date)
```

```
## [1] "Date"
```

Create a new variable named weekpart


```r
imputeNADate$weekpart <- sapply(imputeNADate$date, date2weekpart)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). .

```r
AverageNumberSteps <- ddply(imputeNADate, .(interval, weekpart), summarise, mean = mean(steps))

p <- ggplot(AverageNumberSteps, aes(x=interval, y = mean))
p <- p + geom_line() + facet_grid(. ~weekpart, ) + theme_bw(base_family = "Verdana")
p <- p + ggtitle("Activity pattersns on weekends and weekdays")
p + xlab("Interval")
```

![plot of chunk Create a plot Time Series](figure/Create a plot Time Series-1.png) 
