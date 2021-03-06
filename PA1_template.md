Loading and preprocessing the data
==================================


```r
data1 <- read.csv("activity.csv")
data1[,"date"] <- as.Date(data1[,"date"])
```

What is mean total number of steps taken per day?
=================================================

1) Calculate the total number of steps taken per day

```r
sumofsteps <- tapply(X = data1$steps, INDEX = data1$date, FUN = sum, na.rm = TRUE, simplify = TRUE)
Step1 <- sumofsteps[1:length(sumofsteps)]
```

2) Plot a histogram of total number of steps taken each day

```r
library(ggplot2)
library(plyr)
library(reshape2)
qplot(Step1, geom = "histogram" ,
      binwidth = 1000,
      main = "Histogram for total number of steps taken per day",
      xlab = "total number of steps taken each day", 
      col = I("blue"),
      alpha = I(0.2))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

3) Calculate the mean and median of the total number of steps taken per day

```r
meanstep <- mean(Step1)
medianstep <- median(Step1)
```
The mean of the total number of steps taken per day is 9354.2295082; while the median is 10395.


What is the average daily activity pattern?
===========================================

1) Make a time series plot of average number of steps taken (averaged across all days) against the 5-minute interval.

```r
fivestepmean <- tapply(X = data1$steps, INDEX = data1$interval, FUN = mean, 
                       na.rm = TRUE, simplify = TRUE)
interval <- unique(data1$interval)
fmean <- unname(fivestepmean)
fmean.df <- data.frame(mean1=fmean, interval = interval)

qplot(interval, mean1, data = fmean.df, geom = "line", main = "Time series plot") + 
        ylab("Average number of steps taken") +
        xlab("Interval") 
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

2) Calculate the maximum number of steps

```r
whichmax <- which.max(fmean.df$mean1)
maxsteps <- fmean.df$interval[whichmax]
```
On average across all the days in the dataset, the maximum numer of steps lies in the 835th 5-minute interval. 


Imputing missing values
=======================

1) Calculate the total number of missing values in the dataset

```r
x_na <- is.na(data1$steps)
sum_na <- sum(x_na)
```
There are 2304 missing values in the dataset.

2) To reduce the biasness when doing calculations using the dataset, missing values are replaced by the mean for their respective 5-minute interval.

```r
data2 <- data1
for(i in 1:length(data1[,1])){
        if(is.na(data2$steps[i])){
                k <- fmean.df$mean1[fmean.df$interval == data2$interval[i]]
                data2$steps[i] <- k
        }
}
```

3) data2 is the new dataset with all the missing values filled in with the mean of their respective 5-minute interval.


4) Plot a new histogram

```r
newsum_raw <- tapply(X = data2$steps, INDEX = data2$date, FUN = sum, na.rm = TRUE, simplify = TRUE)
newsum <- unname(newsum_raw)

qplot(newsum, geom = "histogram" ,
      binwidth = 1000,
      main = "New histogram for total number of steps taken per day",
      xlab = "total number of steps taken each day", 
      col = I("blue"),
      alpha = I(0.2))
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

Calculate the new mean and new median

```r
newmean <- mean(newsum)
newmedian <- median(newsum)
```
For the new dataset (data2), the mean of the total number of steps taken per day is 1.0766189 &times; 10<sup>4</sup>; while the median is 1.0766189 &times; 10<sup>4</sup>.
For the old dataset (data1), The mean of the total number of steps taken per day is 9354.2295082; while the median is 10395.
As seen, the mean and median values differ after replacing the missing values with 5-minute interval mean values, and both the mean and median increases greatly. 


Are there differences in activity patterns between weekdays and weekends?
=========================================================================
1) Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
wkday <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
wkdata1 <- weekdays(as.Date(data1$date)) %in% wkday
fdata <- factor(wkdata1, levels = c(TRUE, FALSE), labels = c("weekday", "weekend"))
```

2) Make a panel plot containing a time series plot

```r
data2 <- data.frame(data2, wkk = fdata)
fmean2.df <- aggregate(data2$steps, list(as.numeric(data2$interval),data2$wkk), FUN = "mean")
names(fmean2.df) <- c("interval", "wkk", "mean2")

g <- ggplot(fmean2.df, aes(interval, mean2)) +
        geom_line() +
        ylab("Average number of steps taken") +
        xlab("Interval") +
        labs(title = "Time series plot (panel plot)")
g + facet_grid(wkk ~ .)
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
