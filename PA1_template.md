    Sys.setlocale("LC_TIME", "English")

    ## [1] "English_United States.1252"

1.Code for reading in the dataset and/or processing the data

Unzip and load the data

    unzip("Activity.zip")
    activity <- read.csv("activity.csv")

Classes of data set columns

    str(activity)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    dim(activity)

    ## [1] 17568     3

    names(activity)

    ## [1] "steps"    "date"     "interval"

date needs to be converted:

convert date info in format 'yyyy-mm-dd'

    activity$date <- as.Date(activity$date, "%Y-%m-%d")
    activity$date

Summary of data set:

    summary(activity)

    ##      steps             date               interval     
    ##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
    ##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
    ##  Median :  0.00   Median :2012-10-31   Median :1177.5  
    ##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
    ##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
    ##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
    ##  NA's   :2304

Calculate the total number of steps taken per day:

    stepsPerDay <- aggregate(steps ~ date, activity, sum, na.rm=TRUE)
    sum(stepsPerDay$steps)

    ## [1] 570608

    sum(activity[!is.na(activity$steps),]$steps)

    ## [1] 570608

2.Histogram of the total number of steps taken each day

    hist(stepsPerDay$steps, main = "Total steps by day", xlab = "day", col = "green")

![](PA1_template_md_file__files/figure-markdown_strict/unnamed-chunk-7-1.png)
<https://github.com/xetaro/RepData_PeerAssessment1/blob/master/plot_1.png>

3.Mean and median number of steps taken each day

    mean(stepsPerDay$steps)

    ## [1] 10766.19

    median(stepsPerDay$steps)

    ## [1] 10765

4.Time series plot of the average number of steps taken

Make a time series plot of the 5-minute interval (x-axis) and the
average number of steps taken, averaged across all days (y-axis)

    avgStepsPerInterval <- aggregate(steps ~ interval, activity, mean)

    plot(avgStepsPerInterval, type = "l", xlab = "5-min interval", 
        ylab = "Steps", main = "Average number of steps taken",col= "red")

![](PA1_template_md_file__files/figure-markdown_strict/unnamed-chunk-9-1.png)
<https://github.com/xetaro/RepData_PeerAssessment1/blob/master/plot_2.png>

5.The 5-minute interval that, on average, contains the maximum number of
steps

    avgStepsPerInterval[avgStepsPerInterval$steps==max(avgStepsPerInterval$steps),]

    ##     interval    steps
    ## 104      835 206.1698

Interval 835 with approx. 206 steps contians the maximum number of
steps.

1.  Code to describe and show a strategy for imputing missing data

Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NAs):

    table(is.na(activity$steps))

    ## 
    ## FALSE  TRUE 
    ## 15264  2304

    table(is.na(activity$date))

    ## 
    ## FALSE 
    ## 17568

    table(is.na(activity$interval))

    ## 
    ## FALSE 
    ## 17568

2304 values are NA (all for steps).

Strategy for filling the NA's: mean for that 5-minute interval

    activityFilled <- merge(x = activity, y = avgStepsPerInterval, by="interval", all.x=T)

    activityFilled[is.na(activityFilled$steps.x),]$steps.x <-activityFilled[is.na(activityFilled$steps.x),]$steps.y 

    head(activityFilled)

    ##   interval  steps.x       date  steps.y
    ## 1        0 1.716981 2012-10-01 1.716981
    ## 2        0 0.000000 2012-11-23 1.716981
    ## 3        0 0.000000 2012-10-28 1.716981
    ## 4        0 0.000000 2012-11-06 1.716981
    ## 5        0 0.000000 2012-11-24 1.716981
    ## 6        0 0.000000 2012-11-15 1.716981

    activityFilled <- activityFilled[,c(1:3)]

    names(activityFilled) <- c("interval", "steps", "date")

    head(activityFilled)

    ##   interval    steps       date
    ## 1        0 1.716981 2012-10-01
    ## 2        0 0.000000 2012-11-23
    ## 3        0 0.000000 2012-10-28
    ## 4        0 0.000000 2012-11-06
    ## 5        0 0.000000 2012-11-24
    ## 6        0 0.000000 2012-11-15

1.  Histogram of the total number of steps taken each day after missing
    values are imputed

<!-- -->

    stepsPerDayFilled <- aggregate(steps ~ date, activityFilled, sum)

    hist(stepsPerDayFilled$steps, main = "Total steps by day", xlab = "day", col = "orange")

![](PA1_template_md_file__files/figure-markdown_strict/unnamed-chunk-14-1.png)
<https://github.com/xetaro/RepData_PeerAssessment1/blob/master/plot_3.png>

    mean(stepsPerDayFilled$steps)

    ## [1] 10766.19

    median(stepsPerDayFilled$steps)

    ## [1] 10766.19

Data for comparison from earlier:

    mean(stepsPerDay$steps)

    ## [1] 10766.19

    median(stepsPerDay$steps)

    ## [1] 10765

Mean does not change, but the median does a little bit (1 step), but the
histogram shows, that the data is more centered.

1.  Panel plot comparing the average number of steps taken per 5-minute
    interval across weekdays and weekends

Create a new factor variable in the dataset with two levels - "weekday"
and "weekend" indicating whether a given date is a weekday or weekend
day.

    activityFilled$wd <- factor(format(activity$date, "%A"))

    levels(activityFilled$wd)

    ## [1] "Friday"    "Monday"    "Saturday"  "Sunday"    "Thursday"  "Tuesday"  
    ## [7] "Wednesday"

    levels(activityFilled$wd) <- list(Weekday = c("Friday","Monday","Thursday","Tuesday","Wednesday"), Weekend = c("Saturday","Sunday"))
    levels(activityFilled$wd)

    ## [1] "Weekday" "Weekend"

    table(activityFilled$wd)

    ## 
    ## Weekday Weekend 
    ##   12960    4608

Make a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis)

    avgStepsPerIntervalWithWeekday <- aggregate(steps ~ interval + wd, activityFilled, mean)

    library(lattice)

    xyplot(steps ~ interval | factor(wd), data=avgStepsPerIntervalWithWeekday, type = "l", layout=c(1,2))

![](PA1_template_md_file__files/figure-markdown_strict/unnamed-chunk-18-1.png)
<https://github.com/xetaro/RepData_PeerAssessment1/blob/master/plot_4.png>

We can see at the graph above that activity on the weekday has the
greatest peak from all steps intervals. But, we can see too that
weekends activities has more peaks over a hundred than weekday. This
could be due to the fact that activities on weekdays mostly follow a
work related routine, where we find some more intensity activity in
little a free time that the employ can made some sport. In the other
hand, at weekend we can see better distribution of effort along the
time.
