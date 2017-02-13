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

## original data adjusted R-squared
> summary(m)$adj.r.squared
[1] 0.9442322

## plot original residual distribution
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

Now the skewness is close to 0, and the adjusted R-squared is near 98%, the Box-Cox transformation reduced non-normality in the data and produces a better regression model. 
