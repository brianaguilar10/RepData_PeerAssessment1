------------------------------------------------------------------------

## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a **Fitbit**, **Nike
Fuelband**, or **Jawbone Up**. These type of devices are part of the
“quantified self” movement – a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

-   **Datase**t: *Activity monitoring data* \[52K\]

The variables included in this dataset are:

-   **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as **NA**)

-   **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

-   **interval**: Identifier for the 5-minute interval in which
    measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

------------------------------------------------------------------------

## Loading and Preprocessing the Data

Upon initialization, the necessary libraries especially for plotting
purposes and the dataset was imported into the workspace. The dataset
was assigned to the activity variable as shown:

``` r
library(ggplot2)
library(gridExtra)
activity <- read.csv('activity.csv')
head(activity)
```

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

``` r
summary(activity)
```

    ##      steps            date              interval     
    ##  Min.   :  0.00   Length:17568       Min.   :   0.0  
    ##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
    ##  Median :  0.00   Mode  :character   Median :1177.5  
    ##  Mean   : 37.38                      Mean   :1177.5  
    ##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
    ##  Max.   :806.00                      Max.   :2355.0  
    ##  NA's   :2304

Since the dataset contains *NA* values particularly in the steps column,
these rows were filtered out for the first part of the analysis.

``` r
activity$date <- as.Date(activity$date)
df <- activity[!is.na(activity$steps),]
```

------------------------------------------------------------------------

## What is the mean total number of steps taken per day?

The total number of steps taken per day were calculated by aggregating
the steps everyday using the **sum()** function.

``` r
sum_step <- aggregate(steps ~ date, data = df, sum)
head(sum_step)
```

    ##         date steps
    ## 1 2012-10-02   126
    ## 2 2012-10-03 11352
    ## 3 2012-10-04 12116
    ## 4 2012-10-05 13294
    ## 5 2012-10-06 15420
    ## 6 2012-10-07 11015

After aggregating the total number of steps per day, this was then
plotted into a histogram in order to visualize the distribution of steps
everyday. For this plot, the number of bins set was 30 bins.

``` r
ggplot(sum_step, aes(x=steps)) + 
  geom_histogram(color="black", fill="blue") + 
  labs(title = 'Distribution of Total Steps per Day')
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-4-1.png)

Additionally, the mean and median of the total number of steps taken per
day was calculated for an overview of the recorded number of steps.

``` r
mean(sum_step$steps)
```

    ## [1] 10766.19

``` r
median(sum_step$steps)
```

    ## [1] 10765

After the calculation, it was found out that the mean and the median of
the total number of steps taken per day is **10766.19** and **10765**,
respectively. From the results, the value of the mean and the median is
relatively close to each other.

------------------------------------------------------------------------

## What is the average daily activity pattern

To identify the average daily activity pattern, the number of steps was
aggregated by on the interval every day and then plotted into a line
graph to observe the on-going trend and pattern in one interval averaged
across all days.

``` r
time_int <- aggregate(steps ~ interval, data = df, mean)
ggplot(time_int, aes(interval,steps, group = 1)) +
  geom_line() +
  labs(title='Average Daily Pattern')
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-6-1.png)

Based on the figure, the day where it contains the maximum number of
steps can be found on the highest spike on the graph which corresponds
on the interval **835** with a average step of **206.1698113**.

``` r
time_int[time_int$steps == max(time_int$steps),]
```

    ##     interval    steps
    ## 104      835 206.1698

------------------------------------------------------------------------

## Imputing missing values

The dataset used contains missing values particularly in the steps
column. Previous analyses ignored the missing values first but for this
section, imputation will be done to complete the dataset. First,
identify the number of missing values (NA) in the steps column:

``` r
na_val <- nrow(activity[is.na(activity$steps),])
na_val
```

    ## [1] 2304

It can be seen that the number of missing values that were ignored in
previous analyses is **2304 missing values** which makes up 13% of the
dataset. These values will be imputed wherein the mean number of steps
per interval averaged in all days will be used to replace the NA values
of corresponding interval everyday. If the average steps on interval 0
is 1.7169811, then all interval 0 with the missing value will be
replaced by the average. Its implementation will be as follows:

``` r
impute <- merge(x=activity[is.na(activity$steps),], y=time_int, by = "interval", all.x = TRUE, sort = FALSE)
activity[is.na(activity$steps),]$steps <- impute[order(impute$date, decreasing = FALSE),]$steps.y
head(activity)
```

    ##       steps       date interval
    ## 1 1.7169811 2012-10-01        0
    ## 2 0.3396226 2012-10-01        5
    ## 3 0.1320755 2012-10-01       10
    ## 4 0.1509434 2012-10-01       15
    ## 5 0.0754717 2012-10-01       20
    ## 6 2.0943396 2012-10-01       25

Previously at 2012-10-01, there were only missing values in all
intervals but upon imputation, the dataset is now complete with no
missing values. Let us compare the distribution and some key statistics
of the original and imputed dataset to see if there are remarkable
differences on imputation of missing values:

``` r
df$type <- 'original'
activity$type <- 'imputed'
activity <- rbind(df,activity)
sum_step <- aggregate(steps ~ date+type, data = activity, sum)

ggplot(sum_step,aes(x=steps)) + 
  geom_histogram(data=subset(sum_step,type == 'original'),fill = "red", alpha = 0.5) +
  geom_histogram(data=subset(sum_step,type == 'imputed'),fill = "blue", alpha = 0.5) 
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-10-1.png)

From the superimposed histogram, the red color indicates the original
while the blue color indicates the imputed. As we can see, the two
histograms overlay each other aside from one bin where there is a
significant increase. It is due to the fact that most of the means per
interval fall on the same range as that very bin that is why imputing
the mean of the interval means that most of the data imputed fall into
the bin.

``` r
aggregate(steps ~ type, data = sum_step, mean)
```

    ##       type    steps
    ## 1  imputed 10766.19
    ## 2 original 10766.19

``` r
aggregate(steps ~ type, data = sum_step, median)
```

    ##       type    steps
    ## 1  imputed 10766.19
    ## 2 original 10765.00

Additionally, the mean and median key statistics were compared. The mean
remained the same as the original since we have imputed missing values
using the average of the intervals. On the other hand, the median
increased since previous missing values now have one and its value is
the same as the mean.

------------------------------------------------------------------------

## Are there differences in activity patterns between weekdays and weekends?

Using the imputed dataset, we will now divide the data into weekdays and
weekends to analyze the differences in activity patterns if there is
any.

``` r
activity$day <- ifelse(weekdays(activity$date)=="Saturday" | weekdays(activity$date) == "Sunday",'Weekend','Weekday') 
time_int_weekday <- aggregate(steps ~ interval, data = activity[activity$day=="Weekday",], mean)
time_int_weekday$day <- "Weekday"
time_int_weekend <- aggregate(steps ~ interval, data = activity[activity$day=="Weekend",], mean)
time_int_weekend$day <- "Weekend"

g1 <- ggplot(time_int_weekday, aes(x = interval, y = steps)) +
  geom_line() +
  ylim(0,250) +
  labs(title = 'Average Steps in Weekdays')

g2 <- ggplot(time_int_weekend, aes(x = interval, y = steps)) +
  geom_line() +
  ylim(0, 250) +
  labs(title = 'Average Steps in Weekends')

grid.arrange(g1,g2, nrow = 1)
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-12-1.png)

From the figures above, it can be observed that the mean steps per
interval is different from the weekdays and the weekends. It can be
noticed that the peak mean steps can be found on the weekdays. However,
the mean steps on the weekends is slightly higher than that of the
weekdays across most of the intervals.

------------------------------------------------------------------------
