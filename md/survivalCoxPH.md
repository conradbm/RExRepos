---
license: Creative Commons BY-SA
author: Daniel Wollschlaeger
title: "Survival analysis: Cox proportional hazards model"
categories: [Univariate, Survival]
rerCat: Univariate
tags: [Survival, CoxProportionalHazards]
---

Survival analysis: Cox proportional hazards model
=========================

TODO
-------------------------

 - link to survivalKM, survivalParametric, counting process notation for recurrent events and time dependent covariates 

Install required packages
-------------------------

[`survival`](http://cran.r-project.org/package=survival)


```r
wants <- c("survival")
has   <- wants %in% rownames(installed.packages())
if(any(!has)) install.packages(wants[!has])
```


Simulated right-censored event times with Weibull distribution
-------------------------

Time independent covariates with proportional hazards

$T = (-\ln(U) \, b \, e^{-\bf{X} \bf{\beta}})^{\frac{1}{a}}$, where $U \sim \mathcal{U}(0, 1)$


```r
set.seed(123)
N      <- 180
P      <- 3
sex    <- factor(sample(c("f", "m"), N, replace=TRUE))
X      <- rnorm(N, 0, 1)
IV     <- factor(rep(LETTERS[1:P], each=N/P))
IVeff  <- c(0, -1, 1.5)
Xbeta  <- 0.7*X + IVeff[unclass(IV)] + rnorm(N, 0, 2)
weibA  <- 1.5
weibB  <- 100
U      <- runif(N, 0, 1)
eventT <- ceiling((-log(U)*weibB*exp(-Xbeta))^(1/weibA))
obsLen <- 120
censT  <- rep(obsLen, N)
obsT   <- pmin(eventT, censT)
status <- eventT <= censT
dfSurv <- data.frame(obsT, status, sex, X, IV)
```


Fit the Cox proportional hazards model
-------------------------

Assumptions

$$
\begin{equation*}
\begin{array}{rclcl}
\ln \lambda(t) &=& \ln \lambda_{0}(t) + \beta_{1} X_{1} + \dots + \beta_{p} X_{p}      &=& \ln \lambda_{0}(t) + \bf{X} \bf{\beta}\\
    \lambda(t) &=&     \lambda_{0}(t) \, e^{\beta_{1} X_{1} + \dots + \beta_{p} X_{p}} &=& \lambda_{0}(t) \, e^{\bf{X} \bf{\beta}}\\
S(t)           &=& S_{0}(t)^{\exp(\bf{X} \bf{\beta})} &=& \exp\left(-\Lambda_{0}(t) \, e^{\bf{X} \bf{\beta}}\right)\\
\Lambda(t)     &=& \Lambda_{0}(t) \, e^{\bf{X} \bf{\beta}} & &
\end{array}
\end{equation*}
$$


```r
library(survival)
(fitCPH <- coxph(Surv(obsT, status) ~ X + IV, data=dfSurv))
```

```
Call:
coxph(formula = Surv(obsT, status) ~ X + IV, data = dfSurv)

      coef exp(coef) se(coef)     z       p
X    0.493     1.637   0.0937  5.26 1.4e-07
IVB -0.822     0.439   0.2108 -3.90 9.6e-05
IVC  0.377     1.457   0.1934  1.95 5.1e-02

Likelihood ratio test=51.6  on 3 df, p=3.62e-11  n= 180, number of events= 157 
```

```r
summary(fitCPH)
```

```
Call:
coxph(formula = Surv(obsT, status) ~ X + IV, data = dfSurv)

  n= 180, number of events= 157 

       coef exp(coef) se(coef)     z Pr(>|z|)    
X    0.4930    1.6373   0.0937  5.26  1.4e-07 ***
IVB -0.8224    0.4394   0.2108 -3.90  9.6e-05 ***
IVC  0.3767    1.4575   0.1934  1.95    0.051 .  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 

    exp(coef) exp(-coef) lower .95 upper .95
X       1.637      0.611     1.362     1.967
IVB     0.439      2.276     0.291     0.664
IVC     1.457      0.686     0.998     2.129

Concordance= 0.68  (se = 0.027 )
Rsquare= 0.249   (max possible= 1 )
Likelihood ratio test= 51.6  on 3 df,   p=3.62e-11
Wald test            = 51  on 3 df,   p=4.92e-11
Score (logrank) test = 53.3  on 3 df,   p=1.55e-11
```


Assess model fit
-------------------------

### AIC


```r
library(survival)
extractAIC(fitCPH)
```

```
[1]    3 1368
```


### McFadden, Cox & Snell and Nagelkerke pseudo $R^{2}$

Log-likelihoods for full model and 0-model without predictors X1, X2


```r
LLf <- fitCPH$loglik[2]
LL0 <- fitCPH$loglik[1]
```


McFadden pseudo-$R^2$


```r
as.vector(1 - (LLf / LL0))
```

```
[1] 0.03652
```


Cox & Snell


```r
as.vector(1 - exp((2/N) * (LL0 - LLf)))
```

```
[1] 0.2493
```


Nagelkerke


```r
as.vector((1 - exp((2/N) * (LL0 - LLf))) / (1 - exp(LL0)^(2/N)))
```

```
[1] 0.2494
```


### Model comparisons

Restricted model without factor `IV`


```r
fitCPH1 <- coxph(Surv(obsT, status) ~ X, data=dfSurv)
anova(fitCPH1, fitCPH)          # model comparison
```

```
Analysis of Deviance Table
 Cox model: response is  Surv(obsT, status)
 Model 1: ~ X
 Model 2: ~ X + IV
  loglik Chisq Df P(>|Chi|)    
1   -699                       
2   -681  35.9  2   1.6e-08 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```


Estimate survival-function und cumulative baseline hazard
-------------------------

### Survival function

For average pseudo-observation with covariate values equal to the sample means. For factors: means of dummy-coded indicator variables.


```r
library(survival)                # for survfit()
(CPH <- survfit(fitCPH))
```

```
Call: survfit(formula = fitCPH)

records   n.max n.start  events  median 0.95LCL 0.95UCL 
    180     180     180     157      14      11      19 
```

```r

## survival 2.37-2 has a bug in quantile() so this currently doesn't work
# quantile(CPH, probs=c(0.25, 0.5, 0.75), conf.int=FALSE)
```


More meaningful: Estimated survival function for new specific data


```r
dfNew  <- data.frame(sex=factor(c("f", "f"), levels=levels(dfSurv$sex)),
                       X=c(-2, -2),
                      IV=factor(c("A", "C"), levels=levels(dfSurv$IV)))
CPHnew <- survfit(fitCPH, newdata=dfNew)

par(mar=c(5, 4.5, 4, 2)+0.1, cex.lab=1.4, cex.main=1.4)
plot(CPH, main=expression(paste("Cox PH-estimate ", hat(S)(t), " with CI")),
     xlab="t", ylab="Survival", lwd=2)
lines(CPHnew$time, CPHnew$surv[ , 1], lwd=2, col="blue")
lines(CPHnew$time, CPHnew$surv[ , 2], lwd=2, col="red")
legend(x="topright", lwd=2, col=c("black", "blue", "red"),
       legend=c("pseudo-observation", "sex=f, X=-2, IV=A", "sex=f, X=-2, IV=C"))
```

![plot of chunk rerSurvivalCoxPH01](../content/assets/figure/rerSurvivalCoxPH01.png) 


### Cumulative baseline hazard


```r
library(survival)                # for basehaz()
expCoef  <- exp(coef(fitCPH))
Lambda0A <- basehaz(fitCPH, centered=FALSE)
Lambda0B <- expCoef[2]*Lambda0A$hazard
Lambda0C <- expCoef[3]*Lambda0A$hazard
plot(hazard ~ time, main=expression(paste("Cox PH-estimate ", hat(Lambda)[g](t), " per group")),
     type="s", ylim=c(0, 5), xlab="t", ylab="cumulative hazard", lwd=2, data=Lambda0A)
lines(Lambda0A$time, Lambda0B, lwd=2, col="red")
lines(Lambda0A$time, Lambda0C, lwd=2, col="green")
legend(x="bottomright", lwd=2, col=1:3, legend=LETTERS[1:3])
```

![plot of chunk rerSurvivalCoxPH02](../content/assets/figure/rerSurvivalCoxPH02.png) 


Model diagnostics
-------------------------

### Proportional hazards assumption

Test based on scaled Schoenfeld-residuals


```r
library(survival)                      # for cox.zph()
(czph <- cox.zph(fitCPH))
```

```
           rho chisq      p
X      -0.1039 2.073 0.1500
IVB     0.1559 4.006 0.0453
IVC     0.0406 0.265 0.6069
GLOBAL      NA 5.053 0.1680
```


Plot scaled Schoenfeld-residuals against covariates


```r
par(mfrow=c(2, 2), cex.main=1.4, cex.lab=1.4)
plot(czph)
```

![plot of chunk rerSurvivalCoxPH03](../content/assets/figure/rerSurvivalCoxPH03.png) 


### Influential observations


```r
dfbetas <- residuals(fitCPH, type="dfbetas")

par(mfrow=c(2, 2), cex.main=1.4, cex.lab=1.4)
plot(dfbetas[ , 1], type="h", main="DfBETAS for X",    ylab="DfBETAS", lwd=2)
plot(dfbetas[ , 2], type="h", main="DfBETAS for IV-B", ylab="DfBETAS", lwd=2)
plot(dfbetas[ , 3], type="h", main="DfBETAS for IV-C", ylab="DfBETAS", lwd=2)
```

![plot of chunk rerSurvivalCoxPH04](../content/assets/figure/rerSurvivalCoxPH04.png) 


### Linearity of log hazard

Plot martingale-residuals against continuous covariate with a nonparametric LOESS-smoother


```r
resMart <- residuals(fitCPH, type="martingale")
plot(dfSurv$X, resMart, main="Martingale-residuals for X",
     xlab="X", ylab="Residuen", pch=20)
lines(loess.smooth(dfSurv$X, resMart), lwd=2, col="blue")
legend(x="bottomleft", col="blue", lwd=2, legend="LOESS fit", cex=1.4)
```

![plot of chunk rerSurvivalCoxPH05](../content/assets/figure/rerSurvivalCoxPH05.png) 


Predicted hazard ratios
-------------------------

With respect to an average pseudo-observation with covariate values equal to the sample means. For factors: means of dummy-coded indicator variables.


```r
library(survival)
hazRat <- predict(fitCPH, type="risk")
head(hazRat)
```

```
[1] 1.9166 1.5390 1.3211 0.8617 2.2970 0.8735
```


Detach (automatically) loaded packages (if possible)
-------------------------


```r
try(detach(package:survival))
try(detach(package:splines))
```


Get the article source from GitHub
----------------------------------------------

[R markdown](https://github.com/dwoll/RExRepos/raw/master/Rmd/survivalCoxPH.Rmd) - [markdown](https://github.com/dwoll/RExRepos/raw/master/md/survivalCoxPH.md) - [R code](https://github.com/dwoll/RExRepos/raw/master/R/survivalCoxPH.R) - [all posts](https://github.com/dwoll/RExRepos/)
