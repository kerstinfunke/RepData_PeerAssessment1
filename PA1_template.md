## Loading and preprocessing the data

First, we will load the data, format the Date variable and build a new
data frame which aggregates the number of steps per day.

    library(dplyr)

    activity <- read.csv("activity.csv", 
                         header = TRUE, 
                         sep = ",",
                         na.strings = "NA")

    activity$date <- as.Date(activity$date, "%Y-%m-%d")

    activityDay <- aggregate(activity$steps, 
                             by = list(date=activity$date),
                             FUN=sum)
    activityDay<- rename(activityDay, steps = x)

## Mean total number of steps taken per day

Here you can see a histogram of steps taken per day.

    hist(activityDay$steps, 
         xlab="Steps per Day", 
         main="Histogram of Steps per Day")

![](PA1_template.md_files/figure-markdown_strict/histogram-1.png)

Here are the mean and the median of steps taken per day.

    mean(activityDay$steps,na.rm = TRUE)

    ## [1] 10766.19

    median(activityDay$steps,na.rm = TRUE)

    ## [1] 10765

So the mean of the total number of steps taken per day is 10766.19 and
the median is 10765.

## Average daily activity pattern

First, we build a new data frame which shows the average number of steps
per interval across all days.

    activityInterval <- aggregate(activity$steps, 
                                  by = list(interval=activity$interval), 
                                  FUN=mean, na.rm = TRUE)
    activityInterval<- rename(activityInterval, steps = x)

Then, we build a time series plot of the 5-minute interval (x-axis) and
the average number of steps taken, averaged across all days (y-axis)

    plot(activityInterval$interval, activityInterval$steps, 
         type="l", 
         xlab="5-minute interval", 
         ylab="average number of steps taken, averaged across all days", 
         main="Average daily activity pattern")

![](PA1_template.md_files/figure-markdown_strict/plot-1.png)

We then identify which 5-minute interval, on average across all the days
in the dataset, contains the maximum number of steps.

    activityInterval$interval [which.max(activityInterval$steps)]

    ## [1] 835

This shows that on average across all the days in the dataset the
5-minute interval at 8:35am shows the highest number of steps.

## Imputing missing values

First we determine the number of missing values in the original data set

    sum(!complete.cases(activity$steps))

    ## [1] 2304

Overall there are 2304 missing values.

In a second step we develop a strategy to replace missing values: 1)
identifying the missing values 2) replacing missing values with the
overall average per interval

We then set up a new data frame applying this strategy

    activityMissing <- activity
    activityMissing$steps[is.na(activityMissing$steps)] <- mean(activityMissing$steps, na.rm=TRUE) 

As a next step we build a new data frame which aggregates the number of
steps per day using the new data set

    activityDayMissing <- aggregate(activityMissing$steps, 
                                    by = list(date=activityMissing$date), 
                                    FUN=sum)
    activityDayMissing<- rename(activityDayMissing, steps = x)

Then we make a histogram of the total number of steps taken each day
using this new data frame.

    hist(activityDayMissing$steps, 
         xlab="Steps per Day", 
         main="Histogram of Steps per Day - replaced missing values")

![](PA1_template.md_files/figure-markdown_strict/histogram%20NA-1.png)

We then recalculate the mean and median

    mean(activityDayMissing$steps,na.rm = TRUE)

    ## [1] 10766.19

    median(activityDayMissing$steps,na.rm = TRUE)

    ## [1] 10766.19

The results show that the mean stayed the same and the median now equals
the mean.

## Differences in activity patterns between weekdays and weekends

In order to identify potential differences in activity patterns between
weekdays and weekends we set up new variables - first, a variable that
identifies the different weekdays and then we recode those weekdays into
a variable called ‘weekend’ that distinguishes between weekend and
weekday.

    activityMissing$weekdays <- weekdays(activityMissing$date, abbreviate = FALSE)

    activityMissing$weekend<-recode( activityMissing$weekdays, 
                                     "Saturday" = "Weekend", 
                                     "Sunday" = "Weekend", 
                                     .default = "Weekday")

Now we build a data set that shows how many steps participants took on
average per interval on weekdays vs. weekends.

    activityDayMissingInterval <- aggregate(activityMissing$steps, 
                                           by = list(interval=activityMissing$interval, weekend=activityMissing$weekend), 
                                           FUN=mean, na.rm = TRUE)
    activityDayMissingInterval <- rename(activityDayMissingInterval , steps = x)

In a second step we then set up two time series plots looking at the
development throughout the day (using 5-minute intervals) on weekends
(plot 1) and weekdays (plot 2)

    library(lattice)

    xyplot( steps ~ interval| weekend, 
            data =  activityDayMissingInterval, 
            type = "l", 
            layout=c(1,2), 
            ylab="Number of Steps", 
            xlab="Interval")

![](PA1_template.md_files/figure-markdown_strict/plot%20weekend%20weekday-1.png)
