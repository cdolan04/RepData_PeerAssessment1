
# Reproducible Research: Peer Assesment 1
Author: "Chris Dolan"

Date: "November 13, 2015"

## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day

## Data

The data for this assignment can be downloaded from the course web site, the code to do this
is included below.

*steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

*date: The date on which the measurement was taken in YYYY-MM-DD format

*interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Assignment

The assignement instructions from the course website are shown below:

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the GitHub repository created for this assignment. You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

### Loading and preprocessing the data

1. Load the data and librarys used in the analysis

```r
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
tempf = tempfile()
download.file(url = url, destfile = tempf, method = 'auto')
data <- read.csv(unz(tempf, "activity.csv"), header = TRUE)
unlink(tempf)
library(dplyr)
library(ggplot2)
```
2. Process/transform the data into a format suitable for analysis

```r
## reorder the data so interval is the first column
stpdata <- data[,c(3,1,2)]
```

### What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
This dataframe is fairly long for reporting purposes so we will just print the first rows

```r
totalsteps <- aggregate(steps ~ date, stpdata, sum, na.action = na.omit)
head(totalsteps)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```

2. Make a histogram of the total number of steps taken each day

```r
hist(totalsteps[,2],col="red",
     breaks=seq(from=0, to=26000, by=2000),
     xlab="Total number of steps per day", 
     main="Histogram of the total number of steps taken per day")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day

```r
## mean number of steps per day
mean(totalsteps[,2])
```

```
## [1] 10766.19
```

```r
## median number of steps per day
median(totalsteps[,2])
```

```
## [1] 10765
```

### What is the average daily activity pattern?

1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
ints <- factor(stpdata[,1])
aggsteps <- aggregate(stpdata$steps ~ ints, stpdata,  mean)

##plot average of each interval
plot(aggsteps, type="l", 
     col="red", 
     lwd=1, 
     xlab="Interval", 
     ylab="Average number of steps", 
     main="Average number of steps per intervals")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
maxint <- which.max(aggsteps$`stpdata$steps`)
maxactual <- stpdata[maxint,1]
sprintf("Max steps at the %sth interval also known as interval %s", maxint, maxactual)
```

```
## [1] "Max steps at the 104th interval also known as interval 835"
```

### Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
missing <- as.character(sum(is.na(stpdata)))
sprintf("Number of missing values is: %s", missing)
```

```
## [1] "Number of missing values is: 2304"
```

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

I chose to replace each NA with the average for that interval, this was stored as cleansteps.
The code for both 2. and 3. is shown below

```r
stpdata %>% 
group_by(interval) %>%
mutate(steps= replace(steps, is.na(steps), mean(steps, na.rm=TRUE))) -> cleansteps
```


4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
# Calculate total steps per day and make histogram
totalstepsclean <- aggregate(steps ~ date, cleansteps, sum)
hist(totalstepsclean[,2], breaks=seq(from=0, to=26000, by=2000),
     col="green", 
     xlab="Total number of steps", 
     main="Histogram of total number of steps each day\n(with NAs replaced)")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

```r
#Find mean and median perday after sub-ing NAs
mean(totalstepsclean[,2])
```

```
## [1] 10766.19
```

```r
median(totalstepsclean[,2])
```

```
## [1] 10766.19
```

By replacing the NAs with average values, the median value changed slightly, while the mean value stayed the same as expected since average values were used.


### Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
cleansteps$date <- as.Date(factor(sort(cleansteps$date)))
cleansteps <- cbind(cleansteps, weekdays(cleansteps$date))
names(cleansteps)[names(cleansteps) == 'weekdays(cleansteps$date)'] <- 'Weekdays'

weekd <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')
cleansteps$DayLabel <- factor((cleansteps[,4] %in% weekd), 
                   levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))

daylabelavg <- aggregate(steps ~ interval + DayLabel, cleansteps, mean)
```

2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
ggplot(daylabelavg, aes(x=interval, y=steps)) + geom_line(color="red") + 
        facet_wrap(~ DayLabel, nrow=2, ncol=1) +
        labs(x="Interval Number", y="Number of steps") 
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 








