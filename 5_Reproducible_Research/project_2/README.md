## Introduction
Storms and other severe weather events can cause both public health and economic problems for communities and municipalities. Many severe events can result in fatalities, injuries, and property damage, and preventing such outcomes to the extent possible is a key concern.

This project involves exploring the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.

## Data
The data for this assignment come in the form of a comma-separated-value file compressed via the bzip2 algorithm to reduce its size. You can download the file from the course web site:

- Storm Data [47Mb]

There is also some documentation of the database available. Here you will find how some of the variables are constructed/defined.

- National Weather Service Storm [Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)

- National Climatic Data Center Storm Events FAQ

The events in the database start in the year 1950 and end in November 2011. In the earlier years of the database there are generally fewer events recorded, most likely due to a lack of good records. More recent years should be considered more complete.

## Loading and preprocessing the data
Unzip data to obtain a csv file.

```R
library("data.table")

fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = paste0(getwd(), '/repdata%2Fdata%2Factivity.zip'), method = "curl")
unzip("repdata%2Fdata%2Factivity.zip",exdir = "data")
```

## Reading csv Data into Data.Table. 
```R
activityDT <- data.table::fread(input = "data/activity.csv")
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day

```R
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = TRUE)), .SDcols = c("steps"), by = .(date)] 

head(Total_Steps, 10)

#          date steps
# 1: 2012-10-01     0
# 2: 2012-10-02   126
# 3: 2012-10-03 11352
# 4: 2012-10-04 12116
# 5: 2012-10-05 13294
# 6: 2012-10-06 15420
# 7: 2012-10-07 11015
# 8: 2012-10-08     0
# 9: 2012-10-09 12811
# 10: 2012-10-10  9900
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day. 

```R
library(ggplot2)

png("hist1.png", width=480, height=480)

ggplot(Total_Steps, aes(x = steps)) +
    geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = "Daily Steps", x = "Steps", y = "Frequency")

dev.off()
```

3. Calculate and report the mean and median of the total number of steps taken per day

```R
Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```R
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 

ggplot(IntervalDT, aes(x = interval , y = steps)) +
    geom_line(color="blue", size=1) +
    labs(title = "Avg. Daily Steps", x = "Interval", y = "Avg. Steps per day")
```

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```R
IntervalDT[steps == max(steps), .(max_interval = interval)]
```

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

```R
activityDT[is.na(steps), .N ]

# alternative solution
nrow(activityDT[is.na(steps),])
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```R
# Filling in missing values by mean of dataset. 
activityDT[is.na(steps), "steps"] <- round(activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps")])
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```R
data.table::fwrite(x = activityDT, file = "data/tidyData.csv", quote = FALSE)
```

4. Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```R

# total number of steps taken per day
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = TRUE)), .SDcols = c("steps"), by = .(date)] 

# mean and median total number of steps taken per day
Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]

library(ggplot2)
ggplot(Total_Steps, aes(x = steps)) +
    geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

Type of Estimate | Mean_Steps | Median_Steps
--- | --- | ---
First Part (with na) | 9354.23 | 10395
Second Part (fillin in na with mean) | 10751.74 | 10656

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```R
activityDT[, dateTime := as.POSIXct(date, format = "%Y-%m-%d")]
activityDT[, `Day of Week`:= weekdays(x = dateTime)]

activityDT[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `Day of Week`), "weekday or weekend"] <- "weekday"
activityDT[grepl(pattern = "Saturday|Sunday", x = `Day of Week`), "weekday or weekend"] <- "weekend"
activityDT[, `weekday or weekend` := as.factor(`weekday or weekend`)]
```

2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
