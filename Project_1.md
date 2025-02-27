---
title: "Reproducible Research - Peer Assignment 1"
author: "Clifton Bell"
date: "2023-Nov-19"
output: html_document
keep_md: true
---





## Loading and Processing Data


```r
activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

The following code calculates the total number of steps per day.


```r
activity %>%
  group_by(date) %>%
  summarise_at(vars(steps), list(name = sum)) -> steps_by_date

colnames(steps_by_date)[2] ="daily_step_sum"
```

The following code makes a histogram of the steps per day, and plots it both
to the screen and to a png file named hist_steps1.png.


```r
hist_steps1 <- ggplot(steps_by_date, aes(x=daily_step_sum)) + 
  geom_histogram()
hist_steps1
```

![plot of chunk hist_steps](figure/hist_steps-1.png)

```r
png(here("hist_steps1.png"))
hist_steps1
dev.off()
```

```
## png 
##   2
```

The following code calculates the mean and median number of steps per day.


```r
mean_steps <- mean(steps_by_date$daily_step_sum, na.rm = TRUE)
median_steps <- median(steps_by_date$daily_step_sum, na.rm = TRUE)
```

The mean number of steps per day is 1.0766189 &times; 10<sup>4</sup>, and the median is 10765.

## What is the average daily activity pattern?

The following code makes a time series plot of the 5-minute interval and the
average number of steps taken, averaged across all days.


```r
activity %>%
  group_by(interval) %>%
  summarise_at(vars(steps), list(name = mean), na.rm = TRUE) -> steps_by_int

colnames(steps_by_int)[2] ="mean_steps_by_int"

plot_steps1 <- ggplot(steps_by_int, aes(x=interval, y=mean_steps_by_int)) + 
  geom_point()

row_max <- which.max(steps_by_int$mean_steps_by_int)
max_int <- steps_by_int[row_max, 1]
plot_steps1
```

![plot of chunk daily_pattern](figure/daily_pattern-1.png)

```r
png(here("plot_steps1.png"))
plot_steps1
dev.off()
```

```
## png 
##   2
```

The 5-minute interval (on average across all the days in the dataset) that contains the maximum number of steps is 835.

## Imputing missing values

The following code replaces missing (NA) values of steps with the mean value
for that interval, and plots a new histogram. First calculate and report the
total number of missing values.


```r
no_miss <- sum(is.na(activity$steps))
```

The total number of missing values is 2304.

Next, replace the missing values with the mean value.


```r
activity2 <- activity

my_index <- match(activity2$interval, steps_by_int$interval)

for (i in 1:nrow(activity2))  {
    if (is.na(activity2[i,1])) {activity2[i,1] = steps_by_int[my_index[i],2]} 
}

activity2 %>%
  group_by(date) %>%
  summarise_at(vars(steps), list(name = mean)) -> steps_by_date2

colnames(steps_by_date2)[2] ="daily_step_sum"
```

Create the new histogram (hist_steps2), and calculate the mean and median total number of
steps taken per day.


```r
hist_steps2 <- ggplot(steps_by_date2, aes(x=daily_step_sum)) + 
  geom_histogram()
hist_steps2
```

![plot of chunk new_hist](figure/new_hist-1.png)

```r
png(here("hist_steps2.png"))
hist_steps2
dev.off()
```

```
## png 
##   2
```

```r
new_mean <- mean(steps_by_date2$daily_step_sum, na.rm = TRUE)
new_median <- median(steps_by_date2$daily_step_sum, na.rm = TRUE)
```

With the imputed values, the new median of steps per day is 37.3825996 and
the new median is 37.3825996.

## Are there differences in activity patterns between weekdays and weekends?

The following code creates a new factor variable in the dataset with two levels
– “weekday” and “weekend” -- indicating whether a given date is a weekday or
weekend day. It then makes a panel plot containing a time series plot of the
5-minute interval and the average number of steps taken, averaged across all
weekday days or weekend days.


```r
activity2$date <- ymd(activity2$date)
activity2$day <-weekdays(activity2$date, abbreviate = TRUE)
weekday_list <- c("Mon", "Tue", "Wed", "Thu", "Fri")
weekend_days <- c("Sat", "Sun")

activity2$Day_Type <- ifelse(activity2$day %in% weekday_list, 
                "Weekday", "Weekend")

activity2 %>%
  group_by(interval, Day_Type) %>%
  summarise_at(vars(steps), list(name = mean), na.rm = TRUE) -> steps_by_int2

colnames(steps_by_int2)[3] = "mean_steps_int"

plot_steps2 <- ggplot(steps_by_int2, aes(x=interval, y = mean_steps_int)) +
  geom_line() +
  facet_wrap(~ Day_Type, ncol = 1, scales = "free_y") +
  theme_minimal()
plot_steps2
```

![plot of chunk week](figure/week-1.png)

```r
png(here("plot_steps2.png"))
plot_steps2
dev.off()
```

```
## png 
##   2
```
