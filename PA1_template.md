# Reproducible Research: Peer Assessment 1
### **Created: 2014-7-19**

### **Author: Wacky Young**

## Loading and preprocessing the data

```r
data <- read.csv("activity.csv", header = TRUE)
data$date <- as.Date(data$date)

# create dataframe with total steps per day
data.day <- aggregate(data$steps, by=list(data$date), sum)
names(data.day)[1] <-"day"
names(data.day)[2] <-"steps"

# create dataframe with total steps per interval
data.interval <- aggregate(data$steps, by=list(data$interval), sum, na.rm=TRUE, na.action=NULL)
names(data.interval)[1] <-"interval"
names(data.interval)[2] <-"steps"

# create dataframe with mean steps per interval
data.mean.interval <- aggregate(data$steps, by=list(data$interval), mean, na.rm=TRUE, na.action=NULL)
names(data.mean.interval)[1] <-"interval"
names(data.mean.interval)[2] <-"mean.steps"
```

What is mean total number of steps taken per day?
-------------------------

### Histogram of the total number of steps taken each day

```r
hist(data.day$steps, 
     main = "Histogram of the total number of steps taken each day",
     xlab = "total number of steps taken each day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

### The mean and median total number of steps taken per day

Mean number of steps per day:

```r
mean(data.day$steps, na.rm = TRUE)
```

```
## [1] 10766
```
Median number of steps per day:

```r
median(data.day$steps, na.rm = TRUE )
```

```
## [1] 10765
```

What is the average daily activity pattern?
-------------------------

### Time series plot
_Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)_

```r
plot(data.mean.interval$interval, data.mean.interval$mean.steps, type="n", 
     main="Time Series Plot per 5-minute interval",
     xlab = "5-minute intervals",
     ylab = "Average number of steps taken") 
lines(data.mean.interval$interval, data.mean.interval$mean.steps,type="l") 
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

### Maximum number of steps
_Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?_
5-minute interval with maximum number of steps:

```r
data.mean.interval[which.max(data.mean.interval$mean.steps),1]
```

```
## [1] 835
```
p.s. and the maximum number of steps = 206.1698

Inputing missing values
-------------------------

### Missing values
_Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)_
Total number of missing values in the dataset:

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

### Fill in missing values
_Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc._
I am going to use the mean for the interval as a replacement for missing values. The `data.mean.interval` dataframe (contains mean per interval) has been created during the preprocessing step (see above).

```r
data.missing <- merge(data, data.mean.interval, by = "interval", sort= FALSE) # merge data and data.mean.interval dataframes
data.missing <- data.missing[with(data.missing, order(date,interval)), ] # sort on date and interval (optional)
# replace in steps column NA with value in mean.steps column
data.missing$steps[is.na(data.missing$steps)] <- data.missing$mean.steps[is.na(data.missing$steps)] 
data.missing$mean.steps <- NULL # remove the column with the mean since it is no longer needed
```
Note: the dataset now contains fractions for the number of steps:

```r
head(data.missing)
```

```
##     interval   steps       date
## 1          0 1.71698 2012-10-01
## 63         5 0.33962 2012-10-01
## 128       10 0.13208 2012-10-01
## 205       15 0.15094 2012-10-01
## 264       20 0.07547 2012-10-01
## 327       25 2.09434 2012-10-01
```
The instructions don't list it as a requirement, but it would make sence to round the mean steps since fractions of steps per interval do not make sence. For the purpose of this report I have chosen to round them:

```r
data.missing$steps <- round(data.missing$steps, digits = 0)
```

### New dataset with missing data filled in
_Create a new dataset that is equal to the original dataset but with the missing data filled in._

```r
data.new <- data.missing[, c(2,3,1)]
```

### Histogram of total number of steps
_Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?_

```r
# create dataframe with total steps per day
# different from before since this has NA replaced with mean steps per interval
data.day.new <- aggregate(data.new$steps, by=list(data.new$date), sum)
names(data.day.new)[1] <-"day"
names(data.day.new)[2] <-"steps"
```

### Histogram of the total number of steps taken each day


```r
hist(data.day.new$steps, 
     main = "Histogram of the total number of steps taken each day (NA replaced)",
     xlab = "total number of steps taken each day")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 

### The mean and median total number of steps taken per day

Mean number of steps per day:

```r
# na.rm now is optional since all NA have been replaced!
mean(data.day.new$steps, na.rm = TRUE)
```

```
## [1] 10766
```
Median number of steps per day:

```r
# na.rm now is optional since all NA have been replaced!
median(data.day.new$steps, na.rm = TRUE )
```

```
## [1] 10762
```
The Mean is equal to the estimates from the first part of the assignment.

The Median is slightly lower when compared to the first part of the assignment.  

The histogram shows a similar shape as before with overall higher frequencies due to the NA being replaced in the new histogram. See also this side by side plot:


```r
par(mfrow=c(1,2))

hist(data.day$steps, 
     main = "(with NA)",
     xlab = "total number of steps taken each day")

hist(data.day.new$steps, 
     main = "(NA replaced)",
     xlab = "total number of steps taken each day")
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16.png) 

### Estimates of the total daily number of steps


Are there differences in activity patterns between weekdays and weekends?
-------------------------

### new factor variable
_Create a new factor variable in the dataset with two levels �C ��weekday�� and ��weekend�� indicating whether a given date is a weekday or weekend day._

```r
# create copy of the dataframe
data.new.2 <- data.new
# make sure we use English date names
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
# create a factor with the names of the days for all dates
data.new.2$weekdays <- factor(format(data.new.2$date,'%A'))
# the day names fe
levels(data.new.2$weekdays)
```

```
## [1] "Friday"    "Monday"    "Saturday"  "Sunday"    "Thursday"  "Tuesday"  
## [7] "Wednesday"
```

```r
# replace the levels
levels(data.new.2$weekdays) <- list("weekday" = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"), "weekend" = c("Saturday", "Sunday"))
```

### panel plot
_Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)._

```r
data.new.2.mean.interval <- aggregate(data.new.2$steps, by=list(data.new.2$weekdays, data.new.2$interval), mean, na.rm=TRUE, na.action=NULL)
names(data.new.2.mean.interval)[1] <-"weekday"
names(data.new.2.mean.interval)[2] <-"interval"
names(data.new.2.mean.interval)[3] <-"mean.steps"

library(lattice) 
xyplot(data.new.2.mean.interval$mean.steps ~ data.new.2.mean.interval$interval | data.new.2.mean.interval$weekday, 
       layout=c(1,2), 
       type="l",
       xlab = "Interval",
       ylab = "Number of steps")
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-18.png) 
