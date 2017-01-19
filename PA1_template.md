Loading the data
================

    ## setwd("/Users/tsprabhu/GitHub/RepData_PeerAssessment1-master")
    df <- read.csv("activity.csv", header=TRUE)

Histogram of Steps per Day
==========================

Aggregate the number of steps taken per day and create a histogram of
the aggregated data.

    stepsPerDay <- aggregate(list(Steps=df$step), by=list(Date=df$date), FUN=sum)
    library(ggplot2)
    ggplot(stepsPerDay, aes(x=Steps, col=Date)) + 
        geom_histogram(breaks=seq(0, 22500, by=2500), fill="pink", col="black") +
        ylab("Frequency") +
        ggtitle("Number of Steps Taken per Day") +
         theme(plot.title = element_text(hjust = 0.5))

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](reproducibleReseachProject1_files/figure-markdown_strict/unnamed-chunk-1-1.png)

Mean and median of the total number of steps taken each day
===========================================================

    options(scipen = 999)
    stepsMean <- mean(stepsPerDay$Steps, na.rm=T)
    stepsMedian <- median(stepsPerDay$Steps, na.rm=T) 

The mean of the steps taken per day is **10766.1886792** and the
corresponding median is **10765**.

Average daily activity pattern and the interval with most activity
==================================================================

Take the mean of steps taken during each time interval and create a time
series graph of the averaged data. Also find maximum activity interval

    stepsInterval <- aggregate(list(Steps=df$step), by=list(Interval=df$interval), FUN=mean, na.rm=T)

    # Add column in which time is in time format
    stepsInterval <- transform(stepsInterval, timeOfDay=strptime( paste(formatC(Interval,width=4,flag="0")), "%H%M"))


    library(ggplot2)
    library(scales)
    ggplot(stepsInterval, aes(x=timeOfDay, y=Steps)) + 
        geom_line() +
        xlab("Time of Day") +
         ggtitle("Average Daily Activity") +
         theme(plot.title = element_text(hjust = 0.5))  + 
         scale_x_datetime(labels=date_format("%H:%M", tz="UTC-2"))

![](reproducibleReseachProject1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

    stepsInterval$timeOfDay[which.max(stepsInterval$Steps)]

    ## [1] "2017-01-19 08:35:00 EET"

The interval with most activity on average is **between 08:35 and 08:40
AM**.

Imputing missing values
=======================

    missing <- sum(is.na.data.frame(df))

The number of rows with missing values is **2304**.

Replace the number of steps missing by interval mean.

    x1<-NULL
    modDF <- df
    for (i in 1:length(df$steps)) {
        if(is.na(df$steps[i])==TRUE) {
            x1 <- subset(stepsInterval, Interval == df$interval[i])
            modDF$steps[i] <- x1$Steps
        }
      
    }

Aggregate the number of steps taken per day in the modified data and
create a histogram of the aggregated data.

    stepsPerDayMod <- aggregate(list(Steps=modDF$step), by=list(Date=modDF$date), FUN=sum)
    library(ggplot2)
    ggplot(stepsPerDayMod, aes(x=Steps, col=Date)) + 
        geom_histogram(breaks=seq(0, 22500, by=2500), fill="pink", col="black") +
        ylab("Frequency") +
        ggtitle("Number of Steps Taken per Day, Altered Data") +
         theme(plot.title = element_text(hjust = 0.5))

![](reproducibleReseachProject1_files/figure-markdown_strict/unnamed-chunk-6-1.png)

    options(scipen = 999)
    stepsMeanMod <- mean(stepsPerDayMod$Steps)
    stepsMedianMod <- median(stepsPerDayMod$Steps) 

The mean of the steps taken per day is now **10766.1886792** and the
corresponding median is **10766.1886792**.Replacing the missing values
witht he interval mean does not affect mean steps taken per day, for
obvious reasons. The median was very close to the mean to begin with,
and after replacing NAs the mean and median are equal.

Weekdays vs weekends
====================

Transform dates into appropriate format.

    modDF <- transform(modDF, 
                          date = strptime( paste(date,formatC(interval,width=4,flag="0")), "%Y-%m-%d"))

Create time series graph of mean activity during weekdays and weekends.

    Sys.setlocale("LC_TIME", "C")

    ## [1] "C"

    weekDay <- with(modDF,
                    ifelse(weekdays(date) %in% c("Saturday","Sunday"),"weekend","weekday"))

    modDF$weekDay <- as.factor(weekDay)

    stepsInterval2 <- aggregate(list(Steps=modDF$step), by=list(Interval=modDF$interval, weekDay=modDF$weekDay), FUN=mean)


    # Add column in which time is in time format
    stepsInterval2 <- transform(stepsInterval2, timeOfDay=strptime( paste(formatC(Interval,width=4,flag="0")), "%H%M"))


     library(ggplot2)
     library(scales)
     ggplot(stepsInterval2, aes(x=timeOfDay, y=Steps)) +
         facet_grid(weekDay~.) +
         geom_line() +
         xlab("Time of Day") +
          ggtitle("Average Daily Activity, weekdays and weekends") +
          theme(plot.title = element_text(hjust = 0.5)) +
          scale_x_datetime(labels=date_format("%H:%M", tz="UTF-2"))

![](reproducibleReseachProject1_files/figure-markdown_strict/unnamed-chunk-9-1.png)
