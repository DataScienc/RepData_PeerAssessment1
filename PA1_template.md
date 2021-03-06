# Reproducible Research: Peer Assessment 1
===========================================================================

Hey fellow Data Scientist, below you'll find the completed assignment with notes explaining the code and how i did it.

===========================================================================
## Loading and preprocessing the data
===========================================================================
**WARNING: 'activity.zip' must be located in your working directory for data to load properly.**

### Here I unzip the archive and load the data.

```r
unzip("activity.zip", exdir = "./data")
data <- read.csv("./data/activity.csv")
```


### Next, let's convert the date field from a factor to a date class.

```r
data$date <- as.Date(data$date)
```


=============================================================================
## What is mean total number of steps taken per day?
=============================================================================

### First we make a data.frame of the sum, mean and median of steps grouped by day.

```r
library(plyr)
summary <- ddply(data[complete.cases(data),],
                "date",
                summarise,
                stepssum = sum(steps, na.rm = TRUE),
                mean = round(mean(steps, na.rm = TRUE), 2),
                median = round(median(steps, na.rm = TRUE), 2))
```


### Now let's plot a histogram of the total number of steps taken each day.

```r
hist(summary$stepssum, main = "Histogram of Total Steps Taken Each Day",
     xlab = "Distribution", ylab = "Total Steps Per Day (thousands)")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


### Ok, let's see what the mean and median are for the same variable.

```r
plot(summary$date, summary$mean, type = "l", main = "Mean of Total Steps Per Day", xlab = "Date", ylab = "Steps Per Day")
        points(summary$date, summary$median, type = "l", col = "blue")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 
#### The median is not plotted above since it's 0 for each day.  The data measures steps at 5 minute intervals and since more than 50% of the records are 0 for each day the median will be 0 for each day.

=============================================================================
## What is the average daily activity pattern?
=============================================================================

### First I create a data.frame using the plyr package we called earlier to group by 'interval' across all days and calculate the mean of 'interval'.

```r
x <- ddply(data[complete.cases(data), ],
            "interval",
            summarise,
            avgsteps = round(mean(steps), 2))
```

### Now that our values are organized in 'x' it's easy to plot them here.

```r
plot(x$interval, x$avgsteps, type = "l", xlab = "5-minute interval", ylab = "Average Steps", main = "Average Steps Per 5-Minute Intervals", col = "blue")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 



### Our assignmet asks which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

### We answer this question by finding the 'interval' where 'avgsteps' is equal to the max 'avgsteps' in the data set.

```r
x$interval[x$avgsteps == max(x$avgsteps)]
```

```
## [1] 835
```


=============================================================================
## Imputing missing values
=============================================================================

### This data set contains many missing data points as you can see below.

```r
nrow(data[is.na(data$steps),]) ## Counts the number of rows where steps = NA
```

```
## [1] 2304
```

### We will fill in these missing data points with the average steps for the same interval.

```r
data2 <- merge(data, x, by = "interval")
data2$steps[is.na(data2$steps)] <- data2$avgsteps[is.na(data2$steps)]
data2 <- data2[order(data2$date, data2$interval), ]
```

### Now let's make a new data.frame with the sum, mean and median of steps grouped by day with our new data set.

```r
library(plyr)
summary2 <- ddply(data2[complete.cases(data2),],
                "date",
                summarise,
                stepssum = sum(steps, na.rm = TRUE),
                mean = round(mean(steps, na.rm = TRUE), 2),
                median = round(median(steps, na.rm = TRUE), 2))
```


### Let's plot a histogram of the total number of steps taken each day with our new data set and see if there is any difference.

```r
hist(summary2$stepssum, main = "Histogram of Total Steps Taken Each Day",
     xlab = "Distribution", ylab = "Total Steps Per Day (thousands)")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 


### Looks like it changes the data a bit, let's see what the mean and median are for the same data.

```r
plot(summary2$date, summary2$mean, type = "l", main = "Mean of Total Steps Per Day",
     xlab = "Date", ylab = "Steps Per Day")
        points(summary2$date, summary2$median, type = "l", col = "blue")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 

#### What is the impact of imputing missing data on the estimates of the total daily number of steps?

=============================================================================
## Are there differences in activity patterns between weekdays and weekends?
=============================================================================

### To answer this question let's add a variable that identifies each record by 'weekday' or 'weekend'.

```r
weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
weekends <- c("Saturday", "Sunday")
data2$weekday <- weekdays(data2$date)
data2$weekend <- "weekday" 
data2$weekend[(data2$weekday == weekends)] <- "weekend"
```

### Now we can summarize and plot for 'weekday' and 'weekend' using the 'lattice' package.

```r
x2 <- ddply(data2,
            c("weekend","interval"),
            summarise,
            avgsteps = round(mean(steps), 2))
library(lattice)
xyplot(avgsteps ~ interval | weekend, x2, layout = c(1,2), type = "l")
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

