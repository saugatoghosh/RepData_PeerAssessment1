#Project Assignment 1 - Reproducible Research

##Saugata Ghosh

### August 15, 1947

##Introduction

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals throughout the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

This document presents the results from Project Assignment 1 in the Coursera datascience course on Reproducible Research, written in a single R markdown document that can be processed by knitr and transformed into an HTML file.

Load packages and set default options


```r
library(dplyr)
library(lubridate)
library(lattice)
library(knitr)
```


```r
opts_chunk$set(echo = TRUE, fig.path = 'Figs/')
```
##Loading and preprocessing the data

Show any code that is needed to:
Load the data (i.e. read.csv())
Process/transform the data (if necessary) into a format suitable for your analysis

Reading in the data


```r
data <- read.csv('activity.csv', header =TRUE, stringsAsFactors = FALSE)
```

Change the date into date format using lubridate

```r
data$date <- ymd(data$date)
```
Check the data with str

```r
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```
##What is mean total number of steps taken per day

For this part of the assignment the missing values can be ignored.

1. Calculate the total number of steps taken per day.
2. Make a histogram of the total number of steps taken each day.
3. Calculate and report the mean and median of the total number of steps taken per day.

1. Calculate the total number of steps taken per day removing NAs 

```r
data2 <- aggregate(steps ~ date, data=data, sum, na.rm = TRUE)
```
2. Make histogram


```r
hist(data2$steps, breaks=20, main="Total Steps per Day", xlab="Steps", ylab="Frequency")
```

![plot of chunk unnamed-chunk-7](Figs/unnamed-chunk-7-1.png)

3. Calculate the mean and median of the total number of steps taken per day


```r
meansteps <- mean(data2$steps)
mediansteps <- median(data2$steps)

meansteps
```

```
## [1] 10766.19
```

```r
mediansteps
```

```
## [1] 10765
```

###What is the average daily activity pattern?

To find the average daily activity pattern we aggregated the data on steps by the interval and took the mean. This was then plotted into a graph where the pattern is easily discernible. The maximum value is found numerically and displayed.


```r
data3 <- aggregate(steps ~ interval, data = data, mean, na.rm = TRUE)

plot(data3$interval, data3$steps, type="l", col = "blue",main="Average Steps per Five Minute Interval", xlab="Interval No.", ylab="steps")
```

![plot of chunk unnamed-chunk-9](Figs/unnamed-chunk-9-1.png)


```r
maxsteps <- max(data3$steps)
print(paste("The maximum number of steps in a five minute interval was: ", maxsteps))
```

```
## [1] "The maximum number of steps in a five minute interval was:  206.169811320755"
```

###Imputing missing values

1.Calculate and report the total number of missing values in the dataset


```r
missingdata <- sum(is.na(data))
print(paste("There are", missingdata, "missing data points."))
```

```
## [1] "There are 2304 missing data points."
```

2. Missing values for steps imputed with the mean for that 5-minute interval and new dataset created


```r
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))

datanew <- data %>%
    group_by(interval) %>%
    mutate(
        steps = impute.mean(steps)
    )
```
3. Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day


```r
datanew2 <- aggregate(steps ~ date, data=datanew, sum)

hist(datanew2$steps, breaks=20, main="Total Steps per Day", xlab="Steps", ylab="Frequency")
```

![plot of chunk unnamed-chunk-13](Figs/unnamed-chunk-13-1.png)



```r
meansteps2 <- mean(datanew2$steps)
mediansteps2 <- median(datanew2$steps)

meansteps2
```

```
## [1] 10766.19
```

```r
mediansteps2
```

```
## [1] 10766.19
```
Since we have imputed missing values for steps with mean steps for that 5-minute interval, there is no change in the overall mean steps taken per day while the median steps taken per day is now exactly equal to the mean.

###Are there differences in activity patterns between weekdays and weekends

In order to quickly determine if there are activity differences between weekends and weekdays we simply plot two graphs, one with weekday data and one with weekend data.


```r
datanew$day <- as.factor(ifelse(weekdays(datanew$date) == "Saturday" | weekdays(datanew$date) == "Sunday", "weekend", "weekday"))
```


```r
plotdata <- aggregate(steps ~ interval + day, datanew, mean)
xyplot(steps ~ interval | factor(day), data=plotdata, aspect=1/3, type="l")
```

![plot of chunk unnamed-chunk-16](Figs/unnamed-chunk-16-1.png)




