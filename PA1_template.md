---
title: "Reproducible Research: Peer Assessment 1"
output: html_document
---

## Loading and preprocessing the data

1. Load the data


```r
data <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

Calculate the total number of steps taken per day


```r
library(ggplot2)
total.steps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
```

If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
qplot(total.steps, binwidth=1000, xlab="total number of steps taken each day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

Calculate and report the mean and median of the total number of steps taken per day


```r
mean(total.steps, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(total.steps, na.rm=TRUE)
```

```
## [1] 10395
```


## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
steps_mean <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval),
                      FUN=mean, na.rm=TRUE)
ggplot(data=steps_mean, aes(x=interval, y=steps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
maximum_number_steps <- which.max(steps_mean$average_steps)
maximum_number_steps
```

```
## integer(0)
```


## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I'm using the mean for the 5-minute interval.


```r
mean_interval <- aggregate(data$steps ~ data$interval, data, FUN=mean, na.rm=T)  # calculate the mean 
names(mean_interval) <- c("interval","steps")
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
data_no_nas <- data # new dataset

# loop to replace each NA value with the mean
for (i in 1:nrow(data)) {
    # check if the "steps" value is missing
    if(is.na(data$steps[i])) {
        # if so, replace the missing value with the mean for that interval
        data_no_nas$steps[i] <- mean_interval$steps[which(mean_interval$interval==data$interval[i])]
    }
}
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Both the mean and the median have now a higher value.


```r
library(ggplot2)
total.steps2 <- tapply(data_no_nas$steps, data_no_nas$date, FUN=sum, na.rm=TRUE)
qplot(total.steps2, binwidth=1000, xlab="total number of steps taken each day")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

```r
mean(total.steps2, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(total.steps2, na.rm=TRUE)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
Sys.setlocale("LC_TIME", "English") # set language to english since my pc is set to a different language
```

```
## [1] "English_United States.1252"
```

```r
# function to group the days
daytype <- function(date) {
        if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
                "Weekend"
        } else {
                "Weekday"
        }
  	}

# apply function to column 
data_no_nas$day <- sapply(data_no_nas$date, FUN = daytype)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
averages <- aggregate(steps ~ interval + day, data = data_no_nas, mean)
ggplot(averages, aes(interval, steps)) + geom_line() + facet_grid(day ~ .) + xlab("5-minute interval") + ylab("Number of steps")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
