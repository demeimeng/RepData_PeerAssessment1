Loading and preprocessing the data
----------------------------------

Download and unzip the Activity monitoring data file into the working
directory

1.Load the data

    activity <- read.csv("activity.csv")

2.Process/transform the data

    activity$date<- as.Date(activity$date, format="%Y-%m-%d")

What is mean total number of steps taken per day?
-------------------------------------------------

Ignore the missing values in the dataset.

    steps_per_day <- aggregate(activity$steps ~ activity$date, FUN = "sum", rm.na = TRUE)
    colnames(steps_per_day)<- c("Date", "Steps")

1.Make a histogram of the total number of steps taken each day

    hist(steps_per_day$Steps, xlab = "Steps", main = "Total Number of Steps Per Day")

![](PA1_template_files/figure-markdown_strict/histogram%20total%20steps%20per%20day-1.png)

2.Calculate and report the mean and median total number of steps taken
per day

    mean(steps_per_day$Steps)

    ## [1] 10767.19

    median(steps_per_day$Steps)

    ## [1] 10766

What is the average daily activity pattern?
-------------------------------------------

    average_steps <- aggregate(activity$steps ~ activity$interval, FUN = "mean", rm.na = TRUE)
    colnames(average_steps)<- c("Interval", "Steps")

1.Make a time series plot (i.e. type = "l") of the 5-minute interval
(x-axis) and the average number of steps taken, averaged across all days
(y-axis)

    plot(average_steps, type = "l", main = "Average Steps in 5-Minute Interval")

![](PA1_template_files/figure-markdown_strict/plot%20for%20average%20steps%20per%20interval-1.png)

2.Which 5-minute interval, on average across all the days in the
dataset, contains the maximum number of steps?

    average_steps[which.max(average_steps$Steps), ]$Interval

    ## [1] 835

Imputing missing values
-----------------------

The presence of missing days may introduce bias into some calculations
or summaries of the data.

1.Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NAs)

    nrow(activity[is.na(activity$steps),])

    ## [1] 2304

2.Devise a strategy for filling in all of the missing values in the
dataset. The strategy does not need to be sophisticated. For example,
you could use the mean/median for that day, or the mean for that
5-minute interval, etc.

    # Replace NA with the mean for that 5-minute interval previously calculated

3.Create a new dataset that is equal to the original dataset but with
the missing data filled in.

    activity_imputed <- transform(activity, steps = ifelse(is.na(activity$steps), yes = average_steps$Steps, no = activity$steps))

4.Make a histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day.

    steps_per_day_imputed <- aggregate(activity_imputed$steps ~ activity_imputed$date, FUN = "sum")
    colnames(steps_per_day_imputed)<- c("Date", "Steps")
    hist(steps_per_day_imputed$Steps, xlab = "Steps", main = "Total Number of Steps Per Day", col="Navy")
    hist(steps_per_day$Steps, xlab = "Steps", main = "Total Number of Steps Per Day", col="Light Blue", add = T)
    legend("topright", c("Imputed Data", "Non-NA Data"), fill=c("Navy", "Light Blue") )

![](PA1_template_files/figure-markdown_strict/total%20steps%20per%20day%20with%20filled%20value-1.png)

    mean(steps_per_day_imputed$Steps)

    ## [1] 10766.19

    median(steps_per_day_imputed$Steps)

    ## [1] 10766.19

Do these values differ from the estimates from the first part of the
assignment?

Yes.

What is the impact of imputing missing data on the estimates of the
total daily number of steps?

It increases the frequency near the mean/median value.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Use the dataset with the filled-in missing values for this part.

1.Create a new factor variable in the dataset with two levels --
"weekday" and "weekend" indicating whether a given date is a weekday or
weekend day.

    ## Create new category based on the days of the week
    activity_imputed$weekday <- weekdays(as.Date(activity_imputed$date))
    activity_imputed$category <- ifelse(activity_imputed$weekday %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

2.Make a panel plot containing a time series plot (i.e. type = "l") of
the 5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis). The plot
should look something like the following, which was created using
simulated data:

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.4.4

    average_steps_imputed <- aggregate(steps~interval + category, activity_imputed, mean)
    plot<- ggplot(average_steps_imputed, aes(x = interval , y = steps, color = category)) +
           geom_line() +
           labs(title = "Average Steps in 5-Minute Interval by Day Type", x = "Interval", y = "Average number of steps") +
           facet_wrap(~category, ncol = 1, nrow=2)
    print(plot)

![](PA1_template_files/figure-markdown_strict/Average%20Steps%20by%20Day%20Type-1.png)

The plots shows higher average steps in the afternoon during weekends,
compared to those during weekdays.
