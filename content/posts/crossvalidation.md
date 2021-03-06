---
license: Creative Commons BY-SA
author: Daniel Wollschlaeger
title: "Crossvalidation for linear and generalized linear models"
categories: [Univariate]
rerCat: Univariate
tags: [Regression]
---




Install required packages
-------------------------

[`boot`](http://cran.r-project.org/package=boot)


```r
wants <- c("boot")
has   <- wants %in% rownames(installed.packages())
if(any(!has)) install.packages(wants[!has])
```


$k$-fold crossvalidation
-------------------------

### Simulate data
    

```r
set.seed(123)
N  <- 100
X1 <- rnorm(N, 175, 7)
X2 <- rnorm(N,  30, 8)
X3 <- abs(rnorm(N, 60, 30))
Y  <- 0.5*X1 - 0.3*X2 - 0.4*X3 + 10 + rnorm(N, 0, 3)
dfRegr <- data.frame(X1, X2, X3, Y)
```


### Crossvalidation


```r
glmFit <- glm(Y ~ X1 + X2 + X3, data=dfRegr,
              family=gaussian(link="identity"))
```



```r
library(boot)
k    <- 3
kfCV <- cv.glm(data=dfRegr, glmfit=glmFit, K=k)
kfCV$delta
```

```
[1] 10.16 10.04
```


Leave-one-out crossvalidation
-------------------------


```r
LOOCV <- cv.glm(data=dfRegr, glmfit=glmFit, K=N)
```


CVE = mean(PRESS)


```r
LOOCV$delta
```

```
[1] 10.35 10.35
```


Detach (automatically) loaded packages (if possible)
-------------------------


```r
try(detach(package:boot))
```


Get the article source from GitHub
----------------------------------------------

[R markdown](https://github.com/dwoll/RExRepos/raw/master/Rmd/crossvalidation.Rmd) - [markdown](https://github.com/dwoll/RExRepos/raw/master/md/crossvalidation.md) - [R code](https://github.com/dwoll/RExRepos/raw/master/R/crossvalidation.R) - [all posts](https://github.com/dwoll/RExRepos/)
