# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Extract and Download the activity.zip file in "Data" folder of your working directory

1. Load the data:

```r
raw_data <- read.csv("./data/activity.csv")
```

## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

```r
library(ggplot2)
agg_data_by_date <- aggregate(steps~date, raw_data, sum)
ggplot(agg_data_by_date, aes(x=steps)) + geom_histogram(binwidth=1000)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

2. Calculate and report the mean and median total number of steps taken per day

```r
mean(agg_data_by_date$steps)
```

```
## [1] 10766
```

```r
median(agg_data_by_date$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
agg_data_by_interval <- aggregate(steps~interval, raw_data, FUN=mean)
ggplot(agg_data_by_interval, aes(x = interval, y = steps)) + geom_line() 
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
agg_data_by_interval$interval[which.max(agg_data_by_interval[,2])]
```

```
## [1] 835
```

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(raw_data$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
agg_data_by_interval <- aggregate(steps~interval, raw_data, FUN=mean)
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
library(plyr)
merged_data <- merge(raw_data, agg_data_by_interval, by="interval",all=TRUE)
merged_data$steps.x[is.na(merged_data$steps.x)] <- merged_data$steps.y[is.na(merged_data$steps.x)]
merged_data <- merged_data[order(merged_data$date),c(1:3)]
colnames(merged_data)[2] <- c("steps")
row.names(merged_data) <- seq(nrow(merged_data))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
merged_agg_data_by_date <- aggregate(steps~date, merged_data, sum)
ggplot(merged_agg_data_by_date, aes(x=steps)) + geom_histogram(binwidth=1000)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

```r
mean(merged_agg_data_by_date$steps)
```

```
## [1] 10766
```

```r
median(merged_agg_data_by_date$steps)
```

```
## [1] 10766
```

After imputting the missing data, the mean is same as first part of assignment but the median is changed and is now same as mean. 

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
modified_merged_data = within(merged_data,{day = ifelse(weekdays(as.Date(merged_data$date)) %in% c("Saturday","Sunday"),"Weekend","Weekday")})
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
modified_merged_agg_data_by_interval_and_date <- aggregate(steps~interval+day,modified_merged_data,FUN=mean)
ggplot(modified_merged_agg_data_by_interval_and_date, aes(x = interval, y = steps)) + geom_line() + theme(legend.position = "top") + facet_grid(day ~ .) 
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 
