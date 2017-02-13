## Box-Cox Transformation

### Non-Time Series Data

Box and Cox (1964) suggested a family of power transformations that reduces nonnormality of the error in a linear model. We illustrate the necessity and efficiency of Box Cox transformation using the R build-in dataset 'trees'

```
## import libraries
> library(MASS)
> library(moments)

## print data
> trees
   Girth Height Volume
1    8.3     70   10.3
2    8.6     65   10.3
3    8.8     63   10.2
4   10.5     72   16.4
5   10.7     81   18.8
6   10.8     83   19.7
7   11.0     66   15.6
8   11.0     75   18.2
9   11.1     80   22.6
10  11.2     75   19.9
11  11.3     79   24.2
12  11.4     76   21.0
13  11.4     76   21.4
14  11.7     69   21.3
15  12.0     75   19.1
16  12.9     74   22.2
17  12.9     85   33.8
18  13.3     86   27.4
19  13.7     71   25.7
20  13.8     64   24.9
21  14.0     78   34.5
22  14.2     80   31.7
23  14.5     74   36.3
24  16.0     72   38.3
25  16.3     77   42.6
26  17.3     81   55.4
27  17.5     82   55.7
28  17.9     80   58.3
29  18.0     80   51.5
30  18.0     80   51.0
31  20.6     87   77.0

## run linear regression on original data
> m <- lm(Volume ~ Height + Girth, data = trees)

## original data adjusted R-squared
> summary(m)$adj.r.squared
[1] 0.9442322

## histogram and Q-Q plot of residual
> plot(trees$Girth, rstandard(m))
> par(mfrow = c(2, 1))
> hist(m$resid)
> qqnorm(m$resid)
> qqline(m$resid)
```
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/original.png?raw=true)
Adjusted R-squared measures how close the data are to the fitted regression line. Here we have adjusted R-square at 94%, which means the model explains most of the variability of the response data. However, we might be able to do better. We can see the original residual distribution is not entirely normal, to check this, we check its skewness

```
> skewness(m$resid)
[1] 0.3102985
```

This is evidence that the distribution is indeed somewhat skewed. So we perform Box Cox transformation to reduce non-normality in residuals

```
> bc <- boxcox(Volume ~ Height + Girth, data = trees)
```

We obtain a graph of log-likelihood of lambda,
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/lambda.png?raw=true)
maximizing at a little less than 0.5, then the main term of transformation is y^1/2. To retrieve this lambda

```
> trans <- bc$x[which.max(bc$y)]
> trans
[1] 0.3030303
```

Then run linear regression on the transformed data, and plot the residual distribution again

```
> mnew <- lm((Volume ^ trans) ~ Height + Girth, data = trees)

## transformed data adjusted R-squared
> summary(mnew)$adj.r.squared
[1] 0.975946

## histogram and Q-Q plot of residual
> plot(trees$Girth, rstandard(mnew))
> par(mfrow = c(2, 1))
> hist(mnew$resid)
> qqnorm(mnew$resid)
> 
> qqline(mnew$resid)

> skewness(mnew$resid)
[1] -0.0144304
```

![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/transformed.png?raw=true)

We can see the residual distribution is in a more normal form, to confirm, we check its skewness again,

```
> skewness(mnew$resid)
[1] -0.0144304
```

Now the skewness is close to 0, and the adjusted R-squared is near 98%, the Box-Cox transformation reduced non-normality in the data and lead to a better regression model. 

### Time Series Data 
For time series data, if the distribution of error is non-normal, we can perform Box-Cox transformation using the R package "forecast"

```
## import libraries
library(forecast)

## read in Birth data
> births <- scan("http://robjhyndman.com/tsdldata/data/nybirths.dat")
Read 168 items
> birthstimeseries <- ts(births, frequency=12, start=c(1946,1))
> birthstimeseries
        Jan    Feb    Mar    Apr    May    Jun    Jul    Aug    Sep    Oct    Nov    Dec
1946 26.663 23.598 26.931 24.740 25.806 24.364 24.477 23.901 23.175 23.227 21.672 21.870
1947 21.439 21.089 23.709 21.669 21.752 20.761 23.479 23.824 23.105 23.110 21.759 22.073
1948 21.937 20.035 23.590 21.672 22.222 22.123 23.950 23.504 22.238 23.142 21.059 21.573
1949 21.548 20.000 22.424 20.615 21.761 22.874 24.104 23.748 23.262 22.907 21.519 22.025
1950 22.604 20.894 24.677 23.673 25.320 23.583 24.671 24.454 24.122 24.252 22.084 22.991
1951 23.287 23.049 25.076 24.037 24.430 24.667 26.451 25.618 25.014 25.110 22.964 23.981
1952 23.798 22.270 24.775 22.646 23.988 24.737 26.276 25.816 25.210 25.199 23.162 24.707
1953 24.364 22.644 25.565 24.062 25.431 24.635 27.009 26.606 26.268 26.462 25.246 25.180
1954 24.657 23.304 26.982 26.199 27.210 26.122 26.706 26.878 26.152 26.379 24.712 25.688
1955 24.990 24.239 26.721 23.475 24.767 26.219 28.361 28.599 27.914 27.784 25.693 26.881
1956 26.217 24.218 27.914 26.975 28.527 27.139 28.982 28.169 28.056 29.136 26.291 26.987
1957 26.589 24.848 27.543 26.896 28.878 27.390 28.065 28.141 29.048 28.484 26.634 27.735
1958 27.132 24.924 28.963 26.589 27.931 28.009 29.229 28.759 28.405 27.945 25.912 26.619
1959 26.076 25.286 27.660 25.951 26.398 25.565 28.865 30.000 29.261 29.012 26.992 27.897

## fit a linear model without Box-Cox transformation
> fit <- tslm(birthstimeseries ~ trend + season, lambda = NULL)
> summary(fit)

Call:
tslm(formula = birthstimeseries ~ trend + season, lambda = NULL)

Residuals:
    Min      1Q  Median      3Q     Max 
-2.1819 -0.5458 -0.1180  0.4999  5.1607 

Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
(Intercept) 21.465397   0.323663  66.320  < 2e-16 ***
trend        0.036877   0.001747  21.108  < 2e-16 ***
season2     -1.529948   0.414028  -3.695 0.000304 ***
season3      1.442604   0.414039   3.484 0.000642 ***
season4     -0.260772   0.414058  -0.630 0.529755    
season5      0.789637   0.414084   1.907 0.058377 .  
season6      0.307546   0.414117   0.743 0.458815    
season7      1.873312   0.414157   4.523  1.2e-05 ***
season8      1.650150   0.414205   3.984 0.000104 ***
season9      1.128488   0.414260   2.724 0.007188 ** 
season10     1.157254   0.414323   2.793 0.005879 ** 
season11    -0.768908   0.414393  -1.856 0.065423 .  
season12    -0.055213   0.414470  -0.133 0.894197    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1.095 on 155 degrees of freedom
Multiple R-squared:  0.7929,	Adjusted R-squared:  0.7768 
F-statistic: 49.44 on 12 and 155 DF,  p-value: < 2.2e-16

## histogram and Q-Q plot of residual
> par(mfrow = c(2, 1))
> hist(fit$resid)
> qqnorm(fit$resid)
> qqline(fit$resid)

> skewness(fit$resid)
[1] 1.408475
```
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/ts_fit.png?raw=true)

Now we repeat the same process but this time with Box-Cox transformation activated
```
## fit a second linear model with Box-Cox transformation
> lam <- BoxCox.lambda(birthstimeseries)
> transfit <- tslm(birthstimeseries ~ trend + season, lambda = lam)
> summary(transfit)

Call:
tslm(formula = birthstimeseries ~ trend + season, lambda = lam)

Residuals:
     Min       1Q   Median       3Q      Max 
-0.14673 -0.03218 -0.00496  0.02927  0.32398 

Coefficients:
              Estimate Std. Error t value Pr(>|t|)    
(Intercept)  3.8288206  0.0204807 186.948  < 2e-16 ***
trend        0.0023028  0.0001106  20.830  < 2e-16 ***
season2     -0.0998418  0.0261989  -3.811 0.000199 ***
season3      0.0903352  0.0261996   3.448 0.000727 ***
season4     -0.0170041  0.0262007  -0.649 0.517303    
season5      0.0482851  0.0262024   1.843 0.067272 .  
season6      0.0193845  0.0262045   0.740 0.460576    
season7      0.1155235  0.0262070   4.408 1.94e-05 ***
season8      0.1015224  0.0262101   3.873 0.000158 ***
season9      0.0688406  0.0262136   2.626 0.009502 ** 
season10     0.0709760  0.0262175   2.707 0.007547 ** 
season11    -0.0495573  0.0262220  -1.890 0.060636 .  
season12    -0.0044111  0.0262268  -0.168 0.866652    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 0.06932 on 155 degrees of freedom
Multiple R-squared:  0.999,	Adjusted R-squared:  0.9989 
F-statistic: 1.238e+04 on 12 and 155 DF,  p-value: < 2.2e-16

## histogram and Q-Q plot of residual
> par(mfrow = c(2, 1))
> hist(transfit$resid)
> qqnorm(transfit$resid)
> qqline(transfit$resid)

> skewness(transfit$resid)
[1] 1.342264
```
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/ts_transfit.png?raw=true)
The adjusted R-squared has increased from 78% to nearly 100% by adopting Box-Cox transformation, and the skewness of residual is reduced. Finally, we compare the leave-one-out cross-validatioin error of the two models

```
> CV(fit)
        CV        AIC       AICc        BIC      AdjR2 
 1.3130924 45.0875410 47.8326390 88.8230367  0.7768344 
> CV(transfit)
           CV           AIC          AICc           BIC         AdjR2 
 5.257464e-03 -8.823457e+02 -8.796006e+02 -8.386102e+02  9.988769e-01 
```
It is obvious that the transformed model has much smaller CV error. 
