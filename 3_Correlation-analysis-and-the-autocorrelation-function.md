3\_Correlation analysis and the autocorrelation function
================
IO
4/20/2022

-   [Correlation analysis and the autocorrelation
    function](#correlation-analysis-and-the-autocorrelation-function)
    -   [Scatterplots](#scatterplots)
    -   [Covariance and correlation](#covariance-and-correlation)
    -   [Autocorrelation](#autocorrelation)
-   [Autoregression](#autoregression)
    -   [The autoregressive (AR) model](#the-autoregressive-ar-model)
        -   [Simulating a autoregressive
            model](#simulating-a-autoregressive-model)
        -   [Comparing the random walk (RW) and autoregressive (AR)
            models](#comparing-the-random-walk-rw-and-autoregressive-ar-models)
    -   [AR model estimation and
        forecasting](#ar-model-estimation-and-forecasting)

# Correlation analysis and the autocorrelation function

## Scatterplots

<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_1.png" style="width:50.0%" />

Instead of doing a time plot, we can compare the price values lets say
two stocks in a scatter plot by stock A on the y axis and stock B on the
x axis. This allows us to evaluate the relationship between these stock
price.

<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_2.png" style="width:50.0%" />

Financial asset returns: Channges in price as a fraction of the initial
price over a given time (e.g., a week)

Taking the log of a stock option shows us the percentage of
increase/decrease in the returns. If the stock goes from 1 to 2, this is
a %100 increase, while if it goes from 2 to 3, this increase is %50. In
linear analyses, these two have just 1 unit of increase. With the log
analyses, we can see the relative value of the returns, especially
important for long term trend analyses (like years).

``` r
plot.ts(cbind(EuStockMarkets, log(EuStockMarkets)))
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
# getting the daily returns
## Divide all rows except the first and the last one and subtract 1 from the values
returns <- EuStockMarkets[-1,] / EuStockMarkets[-1860,] - 1

# We'll see that this formula does the same thing as diff(log(x))

#convert the returns to time series
returns <- ts(returns, 
              start = c(1991, 130),
              frequency = 260)

# diff(log(EuStockMarkets)) is the log of returns
plot(cbind(returns, diff(log(EuStockMarkets))))
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
colMeans(returns)
```

    ##          DAX          SMI          CAC         FTSE 
    ## 0.0007052174 0.0008609470 0.0004979471 0.0004637479

``` r
apply(returns, #apply in the data
      MARGIN = 2,     #in columns (1 would be rows)
      FUN = var)      #the var function (variance)
```

    ##          DAX          SMI          CAC         FTSE 
    ## 1.056965e-04 8.523711e-05 1.215909e-04 6.344767e-05

``` r
apply(returns,
      MARGIN = 2,
      FUN = sd)
```

    ##         DAX         SMI         CAC        FTSE 
    ## 0.010280879 0.009232394 0.011026827 0.007965405

``` r
par(mfrow = c(2,2))
apply(returns, MARGIN = 2, FUN = hist, main = "", xlab = "Return")
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
par(mfrow = c(2,2))
apply(returns,
      MARGIN = 2,
      FUN = qqnorm,
      main = "")
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
pairs(EuStockMarkets)
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
plot(diff(log(EuStockMarkets)),
     main = "Log Daily Returns")
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
pairs(diff(log(EuStockMarkets)),
      main = "Log Daily Returns")
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## Covariance and correlation

**Covariance** means to what degree two variables sync their movement
(e.g., increase or decrease) in the values. For time series analysis,
this syncronization is investigated at certain time points while for
behavior, it is investigated by enough number of people showing eleveted
scores for an attribute. The covariance coefficient depends on the scale
of the variables (i.e., how big or small the values are).

**Correlation** is a standardized version of covariance which does not
depend on the scale of the measurements in the variables.

> Correlation = cov(stock\_A, stock\_B) / (sd(stock\_A) \* sd(stock\_B))

<figure>
<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_3.png" style="width:50.0%" alt="Covariance can be seen better with a combined time plot while correlation with a scatterplot" /><figcaption aria-hidden="true">Covariance can be seen better with a combined time plot while correlation with a scatterplot</figcaption>
</figure>

## Autocorrelation

Autocorrelation means correlation analysis is done for a large number of
variables in an efficient way. Lag 1 autocorrelation means the
correlation of a stock prices between all consecutive days (today vs
yesterday). It can be used to see if a time series is dependent on its
past.

``` r
# acf() gives the autocorrelation for each lag, plot can be set to F to get only the values
# Blue lines represent %95 confidence intervals

acf(EuStockMarkets[,1], lag.max = 500, plot = T)
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

# Autoregression

## The autoregressive (AR) model

AR model arguably the most widely used time series model. It is like a
simple linear regression but each observation is regressed on the
previous observation. This makes them a simple first order recursion
processes by regressing today’s observation on yesterday’s observation.
Additionally, WN and RW models are included in AR models as special case
models.

> Today = Slope \* Yesterday + Constant + Noise

In R, we use with he mean centered version of the AR recursion.

> (Today - Mean) = Slope \* (Yesterday - Mean) + Noise

> *Y*<sub>*t*</sub> − *μ* = *ϕ* \* (*Y*<sub>*t* − 1</sub> − *μ*) + *ϵ*<sub>*t*</sub>

*ϵ*<sub>*t*</sub>
is the white noise (WN) with a mean zero (non-zero mean WN would be a
RW).

*ϕ*
is the slope parameter valued between -1 and 1.

If the AR process’s slope
*ϕ* = 0
, then
*Y*<sub>*t*</sub> = *μ* + *ϵ*<sub>*t*</sub>
which is a WN process. If it is
*ϕ* ≠ 0
, then
*Y*<sub>*t*</sub>
depends on both the error
*ϵ*<sub>*t*</sub>
and the previous observation
*Y*<sub>*t* − 1</sub>
. Large values of the slope
*ϕ*
lead to greater autocorrelation and negative values of
*ϕ*
result in oscillatory time series.

<figure>
<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_4.png" style="width:80.0%" alt="(1 &amp; 3) phi is ~1 and each observation is close to its neighbors, meaning they show strong persistance (not oscillating too much) in level (2) phi is negative, shows oscillatory patterns, (4) so much dependence, the series quickly diverging downward" /><figcaption aria-hidden="true">(1 &amp; 3) phi is ~1 and each observation is close to its neighbors, meaning they show strong persistance (not oscillating too much) in level (2) phi is negative, shows oscillatory patterns, (4) so much dependence, the series quickly diverging downward</figcaption>
</figure>

<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_5.png" style="width:80.0%" alt="Autocorrelations in the AR process, (1-3) different positive values of phi, the slope, slower autocorrelation decay/dwindle rate when phi is larger, (4) when phi is negative, the autocorrelation is alternating/oscillating but still autocorrelation is decaying/dwindling" />  
If the mean
*μ* = 0
and the slope
*ϕ* = 1
, then
*Y*<sub>*t*</sub> = *Y*<sub>*t* − 1</sub> + *ϵ*<sub>*t*</sub>
, which is Today = Yesterday + Noise. This is a RW process in which the
*Y*<sub>*t*</sub>
is not stationary.

### Simulating a autoregressive model

``` r
# ar is the phi (the slope) of the time series model
AR_50 <- arima.sim(model = list(ar = .5),
                  n = 100)

AR_90 <- arima.sim(model = list(ar = .9),
                   n = 100)

AR_neg75 <- arima.sim(model = list(ar = -.75),
                   n = 100)

plot(cbind(AR_90, AR_50, AR_neg75),
     main = "As the slope (phi) decreases, oscilattion of the process increase \n and autocorrelation (continuity from one obs to the next) decreases")
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

We can check the autocorrelation for AR models with differing slopes,
like the one above. We can see that higher the slope (phi) of the AR
model, higher autocorrelation. Furthermore, the negative phi value
creates an alternating pattern between positive and negative values.

``` r
par(mfrow = c(2,2))
acf(AR_90)
acf(AR_50)
acf(AR_neg75)
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Persistence is defined by a high correlation between an observation and
its lag, while anti-persistence is defined by a large amount of
variation between an observation and its lag. IN the AR model, when the
slope (phi) parameter approaches 1, the persistance increases but the
process reverts to its mean quickly. Also, the ACF decays to zero at a
quick rate, indicating that values far in the past have little impact on
future values of the process.

<figure>
<img src="3_Correlation%20analysis%20and%20the%20autocorrelation%20function_insertimage_6.png" style="width:50.0%" alt="D shows the highest degree of persistence, with a clear downward trend" /><figcaption aria-hidden="true">D shows the highest degree of persistence, with a clear downward trend</figcaption>
</figure>

### Comparing the random walk (RW) and autoregressive (AR) models

RW is a special case of AR model, which has a slope (phi) value of 1. RW
has the attribute of being non-stationary but it exhibits very strong
persistence. The autocovariance function (ACF) decays to zero very
slowly, meaning past values have a long lasting impact on current
values.

``` r
par(mfrow = c(3,2))
# Simulate and plot AR model with slope 0.9 
AR_90 <- arima.sim(model = list(ar = 0.9), n = 200)
ts.plot(AR_90)
acf(AR_90)

# Simulate and plot AR model with slope 0.98
AR_98 <- arima.sim(model = list(ar = 0.98), n = 200)
ts.plot(AR_98)
acf(AR_98)

# Simulate and plot RW model
RW <- arima.sim(model = list(order=c(0,1,0)), n = 200)
ts.plot(RW)
acf(RW) 
```

![](3_Correlation-analysis-and-the-autocorrelation-function_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## AR model estimation and forecasting
