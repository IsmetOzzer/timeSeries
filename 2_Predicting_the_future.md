2\_Predicting\_the\_future
================
IO
4/19/2022

-   [Removing trends in variability via the logarithmic
    transformation](#removing-trends-in-variability-via-the-logarithmic-transformation)
-   [Removing trends in level by
    differencing](#removing-trends-in-level-by-differencing)
-   [Removing seasonal trends with seasonal
    differencing](#removing-seasonal-trends-with-seasonal-differencing)
-   [The white noise (WN) model](#the-white-noise-wn-model)
    -   [Estimating the white noise in a time
        series](#estimating-the-white-noise-in-a-time-series)
-   [The random walk (RW) model](#the-random-walk-rw-model)
    -   [Stimulating a random walk
        model](#stimulating-a-random-walk-model)
    -   [Stimulating a random walk model with a
        drift](#stimulating-a-random-walk-model-with-a-drift)
-   [Stationary processes](#stationary-processes)

## Removing trends in variability via the logarithmic transformation

<figure>
<img src="2_Predicting_the_future_insertimage_1.png" style="width:50.0%" alt="The exponentially growing graph that we are getting the log() of" /><figcaption aria-hidden="true">The exponentially growing graph that we are getting the log() of</figcaption>
</figure>

<figure>
<img src="2_Predicting_the_future_insertimage_2.png" style="width:50.0%" alt="After we take the log(), the relationship becomes more linear" /><figcaption aria-hidden="true">After we take the log(), the relationship becomes more linear</figcaption>
</figure>

## Removing trends in level by differencing

Differencing with the `diff()` function can give us the increments in
trends over time or any type of change in a time series.

``` r
london <- as.xts(x = London2013$Temperature,
                 order.by = lubridate::as_datetime(London2013$Time)) 

periodicity(london)
```

    ## 30 minute periodicity from 2013-01-01 00:20:00 to 2013-12-31 23:50:00

`diff()` as a default gives 1 lag (differences between 1 observations)

``` r
head(diff(london))
```

    ## Warning: timezone of object (UTC) is different than current timezone ().

    ##                     [,1]
    ## 2013-01-01 00:20:00   NA
    ## 2013-01-01 00:50:00  0.0
    ## 2013-01-01 01:20:00  0.0
    ## 2013-01-01 01:50:00 -1.8
    ## 2013-01-01 02:20:00  0.0
    ## 2013-01-01 02:50:00 -0.4

## Removing seasonal trends with seasonal differencing

Sometimes we are not interested in the seasonal fluctuations in the data
but interested in the yearly changes overall. In such cases, we can get
the differences with the lag of 12 in a montly dataset (substracting
each month’s value from the next year’s month value) and 4 in a
quarterly dataset.

Original monthly driver deaths

``` r
MASS::drivers |> 
  as.xts() |> 
  ts.plot()
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Drivers death data with a lag of 12 (same montly data has been
substracted by each year)

``` r
MASS::drivers |> 
  as.xts() |> 
  diff(lag = 12) |> 
  ts.plot()
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## The white noise (WN) model

Simplest example of a stationary process with no clear pattern or a
trend over time. A weak white noise process has:

-   A fixed, constant mean
-   A fixed, constant variance
-   No correlation over time

<figure>
<img src="2_Predicting_the_future_insertimage_3.png" style="width:40.0%" alt="(a) has a upward trend, (b) has a seasonality effect, (c) more variance at the later observations, (d) no trend, a constant variance, and no correlation over time, so a WN model" /><figcaption aria-hidden="true">(a) has a upward trend, (b) has a seasonality effect, (c) more variance at the later observations, (d) no trend, a constant variance, and no correlation over time, so a WN model</figcaption>
</figure>

ARIMA: AutoRegressive, Integrated Moving Average

We can use ARIMA models to specify white noise (WN) models.

``` r
# Stimulate a WN model with 50 obervations
WN <- arima.sim(model = list(order = c(0,0,0)),
                n = 50)
# This series has a default value of mean 0 and sd 1
head(WN)
```

    ## [1]  0.2729796  0.1782963 -0.3193429 -1.0945342 -0.6523409  0.2817019

``` r
ts.plot(WN)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
# Set the mean and sd of a stimulated WN model
WN_1 <- arima.sim(model = list(order =  c(0,   #Autoregressive order
                                          0,   #Order of integration (differencing)
                                          0)), #Moving average order
                  n = 50,
                  mean = 4,
                  sd = 2)

ts.plot(WN_1)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### Estimating the white noise in a time series

``` r
arima(WN_1, 
      order = c(0,0,0))
```

    ## 
    ## Call:
    ## arima(x = WN_1, order = c(0, 0, 0))
    ## 
    ## Coefficients:
    ##       intercept
    ##          4.0207
    ## s.e.     0.2938
    ## 
    ## sigma^2 estimated as 4.317:  log likelihood = -107.51,  aic = 219.02

``` r
mean(WN_1)
```

    ## [1] 4.020735

``` r
var(WN_1)
```

    ## [1] 4.405029

## The random walk (RW) model

A basic time series model and a simple example of an unstable
(non-stationary) process. They are the cumulative sum (1st obs, 1st +
2nd obs, 1st + 2nd + 3rd obs, …) of a mean zero white noise series.
Therefore, first difference (lag 1) gives a WN model. Also, RW models
are ARIMA(0,1,0) models, which 1 indicates the integration is 1. It has:

-   No specified mean or variance
-   Exhibits strong dependence over time (each observation is strongly
    related to its immediate neighbors)
-   Its changes or increments are (similar to) white noise, which means
    the change is stable/stationary

Some RW time series plots:

<img src="2_Predicting_the_future_insertimage_4.png" style="width:60.0%" />

RW recursion:  
Today (Yt) = Yesterday (Yt - 1) + Noise (Et)

The error (Noise of Et) means zero white noise. The variance of this WN
variance of error is the only parameter of RW.

First difference (lag 1) series:  
Yt - Yt-1 = Et (which is diff(Y), which in itself is a WN series)

<figure>
<img src="2_Predicting_the_future_insertimage_5.png" style="width:50.0%" alt="Y is a RW series while diff(Y) becomes a WN series" /><figcaption aria-hidden="true">Y is a RW series while diff(Y) becomes a WN series</figcaption>
</figure>

Random walk with a drift (with a constant of drift, which drifts/trends
the values upwards or downwards over time):  
Today = Constant (c, which works as the slope in the graph) + Yesterday
+ Noise. This type of RW model increases its parameter size to 2, with a
constant c and a WN variance Et. This is because if you are getting the
cumulative sum of a data with a mean of 0, the data is scattered around
+ and - values (the values are dependent on the variance) and therefore
summing the values results in a somewhat stale model, whereas if the
mean is like 5, since the values are (mostly positive) the trend drifts
upward and if the mean is -5, the trend drifts downwards.

The first difference (lag 1) of a RW with a drift is Yt - Yt-1 = WN
series (process) with a mean as the c of the constant (i.e., constant +
noise in a WN model). In another words, unwinds the cumulative sum to
get the actual data/model. Therefore, if we create a WN process with a
mean other than 0 and then get its cumulative sum (`cumsum()`), we get a
RW process/model.

Some RW with a drift plots:

<figure>
<img src="2_Predicting_the_future_insertimage_6.png" style="width:50.0%" alt="(a) with a drift constant of 0, which is no drift, (b) positive drift coefficient/constant therefore a trend up, (c) negative drift coefficient/constant, therefore a trend down, (d) larger positive coefficient with a steeper upward trend." /><figcaption aria-hidden="true">(a) with a drift constant of 0, which is no drift, (b) positive drift coefficient/constant therefore a trend up, (c) negative drift coefficient/constant, therefore a trend down, (d) larger positive coefficient with a steeper upward trend.</figcaption>
</figure>

### Stimulating a random walk model

``` r
RW <- arima.sim(model = list(order = c(0,    #Autoregressive order
                                       1,    #Order of integration (differencing)
                                       0)),  #Moving average order
                n = 100)                     #100 observations

ts.plot(RW)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
RW_diff <- diff(RW)

plot(RW_diff)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

### Stimulating a random walk model with a drift

``` r
RW_drift <- arima.sim(model = list(order = c(0,
                                             1,
                                             0)),
                      n = 100,
                      mean = .5)

plot(RW_drift)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
RW_drift_diff <- diff(RW_diff)

ts.plot(RW_drift_diff)
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

## Stationary processes

Stationary processes have distributional inveriance (stability) over
time. For observed time series, fluctuations appear random and these
random fluctuations behave similarly from one time period to the next.
For example, stocks or returns from interests have different behavior
from the previous year but their mean, sd, or other statistics are
somewhat similar from one year to the next.

-   Weak stationarity: Mean, variance, covariance are constant over
    time.
    -   Mean and variance Yt is the same (constant) for all times (t).
    -   Covariance of Y at the time t (Yt) and Ys is constant for all
        t - s = h, meaning the covariance between these times depend on
        how close these two time points are.
        -   Cov(Y2, Y5) = Cov(Y7, Y10)

Stationary models (the process) can be modeled with relatively fewer
parameters; there is no need for a different mean for the observation at
the time t (Yt), all times have a common mean which is the mean of the
sample.

Many financial time series do not exhibit stationarity but the changes
in the series are often approximately stationary (constant). A
stationary series should show some oscillation around some fixed level
(mean), which is called *mean-reversion*. For example, the inflation
rates do not naturally come down to a specific level due to being
controlled by the monitary policy. But the changes in inflation rates
show a clear mean-reversion to the mean of 0, as policies increase the
rate at times and decrease at others.

<figure>
<img src="2_Predicting_the_future_insertimage_7.png" style="width:50.0%" alt="(top) Inflation rates, (bottom) changes in inflation rates" /><figcaption aria-hidden="true">(top) Inflation rates, (bottom) changes in inflation rates</figcaption>
</figure>

There are many commonly encountered departures (deviances) from
stationarity, including time trends, periodicity, and a lack of mean
reversion.

<figure>
<img src="2_Predicting_the_future_insertimage_8.png" style="width:50.0%" alt="A shows periodicity, B shows mean-reverting (oscillating around a mean), C shows an upward trend" /><figcaption aria-hidden="true">A shows periodicity, B shows mean-reverting (oscillating around a mean), C shows an upward trend</figcaption>
</figure>

WN models are stationary but the RW models are always non-stationary,
both with and without a drift.

``` r
#Basic WN model
WN <- arima.sim(model = list(order = c(0,
                                       0,
                                       0)),
                n = 100)

#Basic RW model
RW <- cumsum(WN)


#WN model with a drift (mean other than 0)
WN_drift <- arima.sim(model = list(order = c(0,
                                             0,
                                             0)),
                      n = 100,
                      mean = 0.4)

#Get the cumsum of the WN model with a drift
RW_drift <- cumsum(WN_drift)
```

Plotting the models comparatively

``` r
plot.ts(cbind(WN, RW, WN_drift, RW_drift))
```

![](2_Predicting_the_future_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
