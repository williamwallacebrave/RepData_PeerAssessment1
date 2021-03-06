# Reproducible Research: Peer Assessment 1

Before starting the analysis set up all global options for the code chunks

```r
opts_chunk$set(fig.width = 10, fig.height = 6)
```


## Loading and preprocessing the data

```r
data <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

Group steps by date for which they were recorded:

```r
per_day <- aggregate(steps ~ date, data, sum)
barplot(per_day$steps, main = "Number of steps per day", xlab = "Day", ylab = "Total Steps Per Day", 
    names.arg = per_day$date, col = "blue", )
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


To gain some extra perspective into the way one should understand the above we 
can take a look at the histogram of steps:

```r
hist(subset(data, steps > 0)$steps, breaks = 25, xlab = "Number of Steps", main = "Distribution of Number of Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


In order to understand total number of steps a little bit more one could 
calculate the mean and median of the series:

```r
mean_steps <- mean(per_day$steps)
median_steps <- median(per_day$steps)
```

Giving the following results:
- **Mean**: 1.0766 &times; 10<sup>4</sup>
- **Median**: 10765 

## What is the average daily activity pattern?

Let's look at the line plot showing daily activity across all dates:

```r
per_interval <- aggregate(steps ~ interval, data, mean)
x <- per_interval$interval
y <- per_interval$steps
plot(x, y, type = "l", xlab = "Interval", ylab = "Average Steps", main = "Averaged Number of Steps per Interval")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


it's clearly visible that the maximum appears at:

```r
subset(per_interval, steps == max(y))$interval
```

```
## [1] 835
```


## Imputing missing values

```r
number_of_nas <- sum(is.na(data$steps))
```

Total number of NAs is 2304.

In will remove NAs that appear in the **steps** column by replacing them with the
mean number of steps for a date.

```r
library(zoo)
```

```
## 
## Attaching package: 'zoo'
## 
## The following objects are masked from 'package:base':
## 
##     as.Date, as.Date.numeric
```

```r

new_data <- data
new_data$steps <- na.aggregate(data$steps, by = "date", FUN = mean)
```


Let's see whether our strategy for filling up the NAs has any serious impact on the
data. First let's have a look at the distribution of total number of steps for each day

```r
new_per_day <- aggregate(steps ~ date, new_data, sum)
barplot(new_per_day$steps, main = "Number of steps per day", xlab = "Day", ylab = "Total Steps Per Day", 
    names.arg = new_per_day$date, col = "blue", )
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


Also as before let's calculate mean and median for the new data set:

```r
new_mean_steps <- mean(new_per_day$steps)
new_median_steps <- median(new_per_day$steps)
```


Giving the following results:
- **Mean**: 1.0766 &times; 10<sup>4</sup>
- **Median**: 1.0766 &times; 10<sup>4</sup>

One can clearly see that the difference is minimal:
- **Mean Diff**: 0
- **Median Diff**: 1.1887


## Are there differences in activity patterns between weekdays and weekends?

Before answering the question of whether there's any difference bettwen weekend
and weekday behaviour one needs to create new factor variable:

```r
new_data$day_type <- weekdays(as.POSIXlt(new_data$date, format = "%Y-%m-%d"))
weekend <- c("Saturday", "Sunday")
weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
new_data[new_data$day_type %in% weekend, ]$day_type <- "weekend"
new_data[new_data$day_type %in% weekdays, ]$day_type <- "weekday"
new_data$day_type <- factor(new_data$day_type)
```


Finally let's have a look at the difference between weekend and weekday behaviour:

```r
library(lattice)
new_per_interval_weekend <- aggregate(steps ~ interval, subset(new_data, day_type == 
    "weekend"), mean)
new_per_interval_weekend$day_type <- "weekend"

new_per_interval_weekday <- aggregate(steps ~ interval, subset(new_data, day_type == 
    "weekday"), mean)
new_per_interval_weekday$day_type <- "weekday"

day_type_data <- rbind(new_per_interval_weekend, new_per_interval_weekday)
xyplot(steps ~ interval | day_type, data = day_type_data, layout = c(1, 2), 
    type = "l")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 

