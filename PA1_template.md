---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.
Data

The data for this assignment can be downloaded from the course web site.

The variables included in this dataset are:

    * steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

    * date: The date on which the measurement was taken in YYYY-MM-DD format

    * interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Loading and preprocessing the data

```
#set global Rmd Options
{r setoptions, echo=TRUE}
```

Load the following packages


```r
library(dplyr)
library(lattice)
```
Load the Activity Data file and clean the data by omitting all rows with NA in them.

```r
#Download activity file and unzip
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
activity <- read.csv(unz(temp, "activity.csv")) 
unlink(temp)
#Eliminate all rows that contain "NAs"
clean_activity <- na.omit(activity)
```

## What is mean total number of steps taken per day?

```r
#Sum the total number of steps/day
steps_per_day <- clean_activity %>% group_by(date) %>% summarise(total_steps = sum(steps,na.rm = TRUE))

#Plot Histogram of steps/day
hist(steps_per_day$total_steps, main = "Histogram: Total Steps per Day", breaks = "Sturges",
     xlab = "Total Steps", col = "light grey",ylim = c(0,35))
```

![](figure-html/histogram_Original_Activity-1.png)<!-- -->

```r
#Calculate mean and median of steps/day
mean(steps_per_day$total_steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(steps_per_day$total_steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Make a time series line plot of the average number of steps taken in each 5 minute interval (x-axis), averaged across all the days (y-axis).

```r
#Calculate Average Steps per interval
avg_steps_per_interval <- group_by(clean_activity, interval) %>%
         summarise(mean_steps = mean(steps,na.rm = TRUE))

plot(avg_steps_per_interval,type = 'l', main = "Average Daily Activity", col ="blue")
```

![](figure-html/time_series_mean_steps_per_day-1.png)<!-- -->

#Find out which time interval is max average

```r
avg_steps_per_interval <- data.frame(avg_steps_per_interval)
max_interval <- avg_steps_per_interval[avg_steps_per_interval $mean_steps == 
                                    max(avg_steps_per_interval$mean_steps),]
print(max_interval, row.names = FALSE)
```

```
##  interval mean_steps
##       835   206.1698
```

## Imputing missing values
Find number of rows that contain NA

```r
na_activity <- subset(activity, is.na(steps))
nrow(na_activity)
```

```
## [1] 2304
```
The strategy I'm using to fill in the missing values is to first calculate a new dataset of mean steps per day by interval using the cleaned version of the original dataset (all NA removed). This new dataset will then be merged with the original dataset to replace any intervals that are NA.

```r
#convert date to weekday 
activity$day <- weekdays(as.Date(activity$date))

#Explain Strategy
#Calculate the mean steps per day by interval for known data points
daily_activity <- activity %>% group_by(day,interval) %>% summarise(mean_steps = mean(steps,na.rm = TRUE))
#Merge daily activity dataset with original activity dataset
imputed_activity <- merge(activity, daily_activity, by = c("day","interval"))
#Replace NA values in original dataset with mean steps per day.
imputed_activity$imputed_steps <- ifelse(is.na(imputed_activity$steps),
                                         imputed_activity$mean_steps, imputed_activity$steps)
```

#Histogram of Imputed Dataset

```r
#Find number of steps/day
imputed_steps_per_day <- imputed_activity %>% 
  group_by(date) %>% summarise(total_steps = sum(imputed_steps,na.rm=TRUE))

#Plot Histogram of steps/day
hist(imputed_steps_per_day$total_steps, main = "Histogram: Total Steps per Day", breaks = "Sturges",
     xlab = "Total Steps", col = "light grey",ylim = c(0,35))
```

![](figure-html/histogram_imputed_data-1.png)<!-- -->

```r
#Calculate mean and median of steps/day
mean(imputed_steps_per_day$total_steps)
```

```
## [1] 10821.21
```

```r
median(imputed_steps_per_day$total_steps)
```

```
## [1] 11015
```
### Comparison of results between Original and Imputed Data Sets

As you can see from the histograms and the table below both the mean and the median have increased primarily due the additional data resulting from filling in the NA values.

Summary of results:

*Original Mean:   10766.19

*Imputed Mean:    10821.21

*Original Median: 10765

*Imputed Median:  11015

## Are there differences in activity patterns between weekdays and weekends?
Create a day_type factor to distinguish weekdays from weekends and  
make a time series panel plot of the median steps.
As you can see from the resulting plot there is more of a spread of activity over the weekends whereas there is a ore concentrated spike of activity between the 500-1000 interval during the week. 


```r
#Final plot:

#set up weekday/weekend factor
imputed_activity$day_type <- factor(
  ifelse(imputed_activity$day != "Saturday" & imputed_activity$day != 
           "Sunday", "weekday","weekend"))

#plot weekday/weekend mean_steps
imputed_avg_steps <- imputed_activity %>% group_by(day_type,interval) %>% 
    summarise(mean_steps = mean(steps,na.rm = TRUE))
xyplot(mean_steps~interval|day_type,data = imputed_avg_steps, type = "l", layout = c(1,2))
```

![](figure-html/Imputed_activity_weekday_vs_weekend-1.png)<!-- -->
