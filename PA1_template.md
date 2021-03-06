# Reproducible Research Project 1
## Loading and Preprocessing the data  
The file zip is downloaded online and unzipped. The data is stored in a variable called `activitydata`


```r
filename <- "activityzip.zip"

if(!file.exists(filename)){
     fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
     download.file(fileURL, destfile=filename, method = "curl")
}
if(!file.exists("activity")){
     unzip(filename)
}

activitydata <- read.csv("activity.csv")
```

## What is the mean total number of steps taken per day? (ignore missing values)  
- Calculate the total number of steps taken per day  
- Make a histogram of the total number of steps taken each day  
- Calculate and report the mean and median of the total number of steps taken per day


```r
daily_steps <- aggregate(steps ~ date, activitydata, sum)
hist(daily_steps$steps, main = "Frequency of Steps taken per Day", xlab = "Daily Steps", breaks=nrow(daily_steps))
```

![](PA1_template_files/figure-html/mean-1.png)<!-- -->

```r
step_mean <- mean(daily_steps$steps)
step_median <- median(daily_steps$steps)
```

## What is the average daily activity pattern?
Created a time series plot of the average steps taken daily in recorded 5-minute time interval.

```r
average_interval_steps <- aggregate(steps ~ interval, activitydata, mean)
plot(average_interval_steps$interval, average_interval_steps$steps, type="l", 
     main="Average Steps During Daily Time Intervals", xlab="5-Minute Time Interval", ylab="Average Steps")
```

![](PA1_template_files/figure-html/daily-1.png)<!-- -->

```r
max_steps_interval <- average_interval_steps[which.max(average_interval_steps$steps),1]
```
The interval that contains the maximum average number of steps is 835

## Imputing Missing Values
Filled in the missing values in the dataset based on the mean of existing values in that time interval.

```r
num_NA <- sum(is.na(activitydata$steps))
complete_data <- transform(activitydata, steps = ifelse(is.na(activitydata$steps), 
                                       average_interval_steps$steps[match(activitydata$interval, average_interval_steps$interval)],
                                       activitydata$steps))
```
In total, there were 2304 missing step values in the raw data that were replaced.  

Created histogram of total number of steps taken each day with the filled data, and calculated the mean and median total number of steps taken per day.

```r
daily_steps_complete <- aggregate(steps ~ date, complete_data, sum)
hist(daily_steps_complete$steps, breaks=nrow(daily_steps_complete), xlab="Daily Steps", main="Total Steps Taken per Day")
```

![](PA1_template_files/figure-html/meancomplete-1.png)<!-- -->

Calculated the mean and median of the new total steps taken per day, and compared it to the missing data.

```r
step_mean_complete <- mean(daily_steps_complete$steps)
step_median_complete <- median(daily_steps_complete$steps)
percent_change_mean <- (step_mean_complete-step_mean)/step_mean
percent_change_median <- (step_median_complete-step_median)/step_median
extra_steps <- sum(daily_steps_complete$steps)-sum(daily_steps$steps)
```
-The imputed step data mean is 1.0766189\times 10^{4}  
-The imputed step data median is 1.0766189\times 10^{4}  
-The percent difference between non-imputed and imputed mean is 0  
-The percent difference between non-imputed and imputed median is 1.1042074\times 10^{-4}  
-There were 8.6129509\times 10^{4} extra steps in the imputed data.

## Are there differences in activity patterns between weekdays and weekends?
Classified the data as occurring on weekdays or weekends. Then, plotted the data to compare average activity throughout the day to compare activities. Weekends have more constant activity throughout the day, while there are generally higher spikes on weekdays.

```r
week_days <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
complete_data$Week_part = as.factor(ifelse(is.element(weekdays(as.Date(complete_data$date)),week_days), "Weekday", "Weekend"))
steps_by_interval_week <- aggregate(steps ~ interval + Week_part, complete_data, mean)

library(lattice)
xyplot(steps_by_interval_week$steps ~ steps_by_interval_week$interval|steps_by_interval_week$Week_part, main="Average Daily Steps During Time Interval",xlab="Interval", ylab="Average Steps",layout=c(1,2), type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->
