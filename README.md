## Box-Cox Transformation

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


## plot original residual distribution
> plot(trees$Girth, rstandard(m))
> par(mfrow = c(2, 1))
> hist(m$resid)
> qqnorm(m$resid)
> qqline(m$resid)
```
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/original.png?raw=true)
We can see the original residual distribution is not entirely normal, to check this, we exam the skewness

```
> skewness(m$resid)
[1] 0.3102985
```

This is evidence that the distribution is indeed somewhat skewed. So we perform Box Cox transformation to reduce non-normality in residuals

```
> bc <- boxcox(Volume ~ Height + Girth, data = trees)
```

We obtain a graph of log-likelihood of lambda, maximizing at a little less than 0.5, 
![original resid dist](https://github.com/xinyix/Box-Cox-and-Durbin-Watson/blob/master/lambda.png?raw=true)
To retrieve this lambda

```
> trans <- bc$x[which.max(bc$y)]
> trans
[1] 0.3030303
```

Then run linear regression on the transformed data, and plot the residual distribution again

```
> mnew <- lm((Volume ^ trans) ~ Height + Girth, data = trees)
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

Now the skewness is close to 0, the transformation was successful.


## Durbin-Watson Statistics
In statistics, the Durbin–Watson statistic is a test statistic used to detect the presence of autocorrelation (a relationship between values separated from each other by a given time lag) in the residuals (prediction errors) from a regression analysis. We use user-generated data to illustrate its usefulness, 

```
## import library
> library(lmtest)
```

First we generate a AR(1) error term with parameter rho = 0 (white noise) and perform the Durbin-Watson test

```
> err1 <- rnorm(100)
> 
> ## generate regressor and dependent variable
> x <- rep(c(-1,1), 50)
> y1 <- 1 + x + err1
> 
> ## perform Durbin-Watson test
> dwtest(y1 ~ x)

	Durbin-Watson test

data:  y1 ~ x
DW = 1.9101, p-value = 0.3621
alternative hypothesis: true autocorrelation is greater than 0
```

Now we generate another AR(1) error term with parameter rho = 0.9, then run Durbin-Watson test again and expect strong autocorelation in the residuals

```
> err2 <- filter(err1, 0.9, method="recursive")
> y2 <- 1 + x + err2
> dwtest(y2 ~ x)

	Durbin-Watson test

data:  y2 ~ x
DW = 0.33352, p-value < 2.2e-16
alternative hypothesis: true autocorrelation is greater than 0
```

Since in the Durbin-Watson test, the null hypothesis is that there is no correlation among residuals, i.e., they are independent, and the alternative hypothesis is that residuals are autocorrelated.A p-value near 0 means we can reject the null (and conclude residuals are autocorelated). This is consistent with our examples, where in the white noise example we obtain p-value much greater from 0, but in the second example p-value is basically 0.
