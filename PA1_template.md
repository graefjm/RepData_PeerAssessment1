# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data 

#### 1. Unzip the data within your working directory from the "activity" zip file. Then read in the data from the given data set "activity.csv".

```r
activity <- read.csv("activity.csv", stringsAsFactors=FALSE)
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
summary(activity)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```

#### 2. Next convert the data to POSIXct class using the lubridate package and the intervals to a hour:minute format.

```r
library(lubridate)
```

```
## Warning: package 'lubridate' was built under R version 3.2.3
```

```r
activity$date <- ymd(activity$date)

str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : POSIXct, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?

#### 1. Calculate total number of steps per day but ignore all of the missing values

```r
require(dplyr)
```

```
## Loading required package: dplyr
```

```
## Warning: package 'dplyr' was built under R version 3.2.3
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:lubridate':
## 
##     intersect, setdiff, union
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
total_day <- activity %>% group_by(date) %>%summarise(total_steps=sum(steps,na.rm=TRUE),na=mean(is.na(steps))) %>% print
```

```
## Source: local data frame [61 x 3]
## 
##          date total_steps    na
##        (time)       (int) (dbl)
## 1  2012-10-01           0     1
## 2  2012-10-02         126     0
## 3  2012-10-03       11352     0
## 4  2012-10-04       12116     0
## 5  2012-10-05       13294     0
## 6  2012-10-06       15420     0
## 7  2012-10-07       11015     0
## 8  2012-10-08           0     1
## 9  2012-10-09       12811     0
## 10 2012-10-10        9900     0
## ..        ...         ...   ...
```
Plot the total steps per day


```r
barplot(height = total_day$total_steps,names.arg=total_day$date,cex.names=0.68,las=3,col="darkcyan")
abline(h=median(total_day$total_steps),lwd=3, col="orangered")
abline(h=mean(total_day$total_steps),lwd=3, col="orangered3")
text(x = 0,y=median(total_day$total_steps),pos=3,labels = "median")
text(x = 0,y=mean(total_day$total_steps),pos=1,labels = "mean")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)

####  2. Make a histogram of the total number of steps per day

The histogram does not show days with all missing values and not real data.

```r
total_day <- filter(total_day, na < 1)
hist(total_day$total_steps,col="darkcyan",breaks=20,main="Total steps taken per day",xlab="Number of steps per day")
abline(v=median(total_day$total_steps), lwd=4, col="orangered")
legend(legend="median","topright",lwd=2,bty = "n")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)

#### 3. Calculate and display the mean and median of total steps per day


```r
mean_steps <- mean(total_day$total_steps,na.rm=TRUE)
median_steps <- median(total_day$total_steps,na.rm=TRUE)
```
The mean and median for total steps taken per day are 1.076619\times 10^{4} steps and 10765 steps.


## What is the average daily activity pattern?

#### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  


```r
library(dplyr,quietly = TRUE)
daily_patterns <- activity %>% group_by(interval) %>% summarise(average=mean(steps,na.rm=TRUE))
plot(x = 1:nrow(daily_patterns),y = daily_patterns$average,type = "l",
     col = "darkcyan", xaxt = "n",xlab="Intervals", 
     ylab = "Average for interval across all days")
axis(1,labels=daily_patterns$interval[seq(1,288,12)],
     at = seq_along(daily_patterns$interval)[seq(1,288,12)])
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_numb_steps_interval <- filter(daily_patterns,average==max(average))
```

Interval **"835"** contains on average the maximum number of steps (**206.17 steps**).

## Imputing missing values

#### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
na_number <- sum(is.na(activity$steps))
na_number
```

```
## [1] 2304
```

```r
percentage_na <- mean(is.na(activity$steps))
percentage_na
```

```
## [1] 0.1311475
```
The total number of missing values is **2304 ** ( **13.1** % of total observations).

#### 2. Devise a strategy for filling in all of the missing values in the dataset

There are a large number of missing values in the dataset. We will impute missing values based on the average number of steps in a particular 5-minutes interval. 

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
without_NAs <- numeric(nrow(activity))
for (i in 1:nrow(activity))
{
        if (is.na(activity[i,"steps"])==TRUE)
            {
                    without_NAs[i]<-filter(daily_patterns,interval==activity[i,"interval"]) %>% select(average)
            } 
        else
            {
                    without_NAs[i]<-activity[i,"steps"]
            }
                    
}
activity_without_NAs<-mutate(activity,steps_no_NAs=without_NAs)
head(activity_without_NAs)
```

```
##   steps       date interval steps_no_NAs
## 1    NA 2012-10-01        0     1.716981
## 2    NA 2012-10-01        5    0.3396226
## 3    NA 2012-10-01       10    0.1320755
## 4    NA 2012-10-01       15    0.1509434
## 5    NA 2012-10-01       20    0.0754717
## 6    NA 2012-10-01       25      2.09434
```

Verify that the process of imputing missing values preserved the original data points

```r
check <- filter(activity_without_NAs,!is.na(steps)) %>% mutate(ok = (steps==steps_no_NAs))
mean(check$ok)
```

```
## [1] 1
```

#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day


```r
total_day_noNAs <- activity_without_NAs %>% mutate(steps_no_NAs=as.numeric(steps_no_NAs)) %>% group_by(date) %>% summarise(total_steps=sum(steps_no_NAs))
hist(total_day_noNAs$total_steps,col="darkcyan",breaks=20,main="Total steps per day",xlab="Steps per day")
abline(v=median(total_day$total_steps), lwd=4, col="orangered")
legend(legend="median","topright",lwd=3,bty = "n")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)


```r
summary(total_day_noNAs$total_steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
```

By imputing missing values the mean of the total number of steps taken per day increased while median decreased. The imputing of missing data resulted in an increase of total daily number of steps as well.


## Are there differences in activity patterns between weekdays and weekends?

#### 1. Create a new factor variable in the dataset with two levels "weekday" and "weekend" indicating whether a given date is a weekday or weekend day


```r
library(lubridate)
is_weekday <-function(date){
        if(wday(date)%in%c(1,7)) result<-"weekend"
        else
                result<-"weekday"
        result
}

activity_without_NAs <- mutate(activity_without_NAs,date=ymd(date)) %>% mutate(day=sapply(date,is_weekday))

table(activity_without_NAs$day)
```

```
## 
## weekday weekend 
##   12960    4608
```

#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)



```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.2.3
```

```r
daily_patterns <- activity_without_NAs %>% mutate(day=factor(day,levels=c("weekend","weekday")),steps_no_NAs=as.numeric(steps_no_NAs)) %>% group_by(interval,day) %>% summarise(average=mean(steps_no_NAs))
qplot(interval,average,data=daily_patterns,geom="line",facets=day~.)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)