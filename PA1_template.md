Introduction
============

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

Data
====

The data for this assignment can be downloaded from the course web site:
- **Dataset**: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)


- **date**: The date on which the measurement was taken in YYYY-MM-DD format


- **interval**: Identifier for the 5-minute interval in which measurement was taken


The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

Assignment
==========

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in **a single R markdown** document that can be processed by **knitr** and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. **This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.**

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the GitHub repository created for this assignment. You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

Loading and preprocessing the data
----------------------------------

Show any code that is needed to

1. Load the data (i.e. read.csv())

**Download the file, unzip and read the csv**


```r
# Create data directory if needed
if (!file.exists("data")) {
    dir.create("data")
}

# Download and unzip the activity.csv file
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = "./data/activity.zip", method = "curl")
unzip("./data/activity.zip", exdir = "./data", overwrite = TRUE)

# Read the activity.csv file
act <- read.csv("./data/activity.csv", header = TRUE)
```



2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
library(lubridate)
library(doBy)
```

```
## Loading required package: survival
## Loading required package: splines
## Loading required package: MASS
```

```r
library(lattice)

# Create data time string
act$datetimestr <- paste(act$date, substr(paste0("000", as.character(act$interval)), 
    nchar(act$interval), nchar(act$interval) + 4))

# Convert date time
act$datetime <- as.POSIXlt(act$datetimestr, format = "%Y-%m-%d %H%M")

# Create interval string (time)
act$intervalstr <- paste(substr(act$datetimestr, 12, 13), substr(act$datetimestr, 
    14, 15), sep = ":")

# Calculate Day of Week, Weekend Logical, and Weekend string
act$DOW <- wday(act$datetime)
act$WE <- ifelse(act$DOW %in% c(1, 7), TRUE, FALSE)
act$weekday <- ifelse(act$WE, "weekend", "weekday")

# Aggregate Total Steps by Date
totStep <- aggregate(steps ~ date, data = act, sum)

# Aggregate Mean Steps by Interval
meanInter <- aggregate(steps ~ interval, data = act, mean)
```


What is mean total number of steps taken per day?
-------------------------------------------------

For this part of the assignment, you can ignore the missing values in the dataset.

1. Make a histogram of the total number of steps taken each day


```r
hist(totStep$steps, breaks = 22, col = "blue", main = "Total Number of Steps Taken Each Day", 
    xlab = "Number of Steps Taken per Day", ylab = "Frequency")
```

![plot of chunk unnamed-chunk-3](./figures/PA-fig-1-unnamed-chunk-3.png) 



2. Calculate and report the **mean** and **median** total number of steps taken per day


```r
# Calculate the mean and median total number of steps
meanStep <- mean(totStep$steps, na.rm = TRUE)
medianStep <- median(totStep$steps, na.rm = TRUE)
meanStep
```

```
## [1] 10766
```

```r
medianStep
```

```
## [1] 10765
```



What is the average daily activity pattern?
-------------------------------------------

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)



```r
plot(meanInter$interval, meanInter$steps, type = "l", col = "blue", main = "Average Number of Steps Taken", 
    xlab = "5 Minute Interval", ylab = "Number of Steps")
```

![plot of chunk unnamed-chunk-5](./figures/PA-fig-2-unnamed-chunk-5.png) 


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# What 5 minitue interval contains maximum number of steps
meanInter[which.max(meanInter$steps), ]
```

```
##     interval steps
## 104      835 206.2
```


Imputing missing values
-----------------------

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
# count all rows where value of steps is NA
sum(is.na(act$steps))
```

```
## [1] 2304
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. **Imputing stragegy is to use the average number of steps for each interval to fill in missing data.**


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
# Merge activity data and average steps per interval
clean <- merge(act, meanInter, by.x = c("interval"), by.y = c("interval"))

# Overwrite the steps column if value is NA
clean$steps <- ifelse(!is.na(clean$steps.x), clean$steps.x, round(clean$steps.y, 
    digits = 0))
totCleanStep <- aggregate(steps ~ date, data = clean, sum)

# Reorganize data frame
clean <- subset(clean, select = c("steps", "date", "interval", "datetimestr", 
    "datetime", "intervalstr", "DOW", "WE", "weekday"))

# Compare Structures
str(act)
```

```
## 'data.frame':	17568 obs. of  9 variables:
##  $ steps      : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date       : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval   : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ datetimestr: chr  "2012-10-01 0000" "2012-10-01 0005" "2012-10-01 0010" "2012-10-01 0015" ...
##  $ datetime   : POSIXlt, format: "2012-10-01 00:00:00" "2012-10-01 00:05:00" ...
##  $ intervalstr: chr  "00:00" "00:05" "00:10" "00:15" ...
##  $ DOW        : num  2 2 2 2 2 2 2 2 2 2 ...
##  $ WE         : logi  FALSE FALSE FALSE FALSE FALSE FALSE ...
##  $ weekday    : chr  "weekday" "weekday" "weekday" "weekday" ...
```

```r
str(clean)
```

```
## 'data.frame':	17568 obs. of  9 variables:
##  $ steps      : num  2 0 0 0 0 0 0 0 0 0 ...
##  $ date       : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 54 28 37 55 46 20 47 38 56 ...
##  $ interval   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ datetimestr: chr  "2012-10-01 0000" "2012-11-23 0000" "2012-10-28 0000" "2012-11-06 0000" ...
##  $ datetime   : POSIXlt, format: "2012-10-01 00:00:00" "2012-11-23 00:00:00" ...
##  $ intervalstr: chr  "00:00" "00:00" "00:00" "00:00" ...
##  $ DOW        : num  2 6 1 3 7 5 7 6 4 1 ...
##  $ WE         : logi  FALSE FALSE TRUE FALSE TRUE FALSE ...
##  $ weekday    : chr  "weekday" "weekday" "weekend" "weekday" ...
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
hist(totCleanStep$steps, breaks = 22, col = "blue", main = "Total Number of Steps Taken Each Day", 
    xlab = "Number of Steps Taken per Day", ylab = "Frequency")
```

![plot of chunk unnamed-chunk-9](./figures/PA-fig-3-unnamed-chunk-9.png) 



```r
# Calculate mean and median of the cleansed data
meanCleanStep <- mean(totCleanStep$steps)
medianCleanStep <- median(totCleanStep$steps)

# Compare mean
meanCleanStep
```

```
## [1] 10766
```

```r
meanStep
```

```
## [1] 10766
```

```r

# Compare median
medianCleanStep
```

```
## [1] 10762
```

```r
medianStep
```

```
## [1] 10765
```


**The median value differs from the original data the mean value is unchanged**
**The total daily number of steps value increases**

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
# Create Plot Data
pd <- subset(clean, select = c("interval", "weekday", "steps"))
names(pd) = c("interval", "weekday", "steps")

# Create factors
pd$interval <- factor(pd$interval, exclude = "")
pd$weekday <- factor(pd$weekday, exclude = "")

# Summarize by interval and weekday
avgStepInter <- summaryBy(steps ~ interval + weekday, data = pd, FUN = mean)
avgStepInter$steps.mean <- round(avgStepInter$steps.mean, digits = 0)
names(avgStepInter) <- c("interval", "weekday", "steps")
```


2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
xyplot(steps ~ interval | weekday, data = avgStepInter, type = c("l", "l"), 
    layout = c(1, 2), xlab = "Interval", xlim = c(0, 288), ylab = "Number of steps", 
    ylim = c(0, 250), scales = list(x = list(at = seq(0, 288, 60), labels = c("0000", 
        "0500", "1000", "1500", "2000", "2359")), y = list(at = seq(0, 250, 
        50), labels = c("0", "50", "100", "150", "200", "250"))), main = "Comparison of the Activity Patterns between Weekends and Weekdays")
```

![plot of chunk unnamed-chunk-12](./figures/PA-fig-4-unnamed-chunk-12.png) 



