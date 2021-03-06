---
license: Creative Commons BY-SA
author: Daniel Wollschlaeger
title: "Split-plot-factorial ANOVA (SPF-p.q design)"
categories: [Univariate, ANOVA]
rerCat: Univariate
tags: [ANOVA]
---

Split-plot-factorial ANOVA (SPF-p.q design)
=========================

TODO
-------------------------

 - link to anovaSPFpqr, anovaMixed, dfReshape

Traditional univariate analysis and multivariate approach.

Install required packages
-------------------------

[`car`](http://cran.r-project.org/package=car), [`multcomp`](http://cran.r-project.org/package=multcomp)


```r
wants <- c("car", "multcomp")
has   <- wants %in% rownames(installed.packages())
if(any(!has)) install.packages(wants[!has])
```


Two-way SPF-$p \cdot q$ ANOVA
-------------------------

### Using `aov()` with data in long format


```r
set.seed(123)
Nj   <- 10
P    <- 3
Q    <- 3
muJK <- c(rep(c(1,-1,-2), Nj), rep(c(2,1,-1), Nj), rep(c(3,3,0), Nj))
dfSPFpqL <- data.frame(id=factor(rep(1:(P*Nj), times=Q)),
                       IVbtw=factor(rep(LETTERS[1:P], times=Q*Nj)),
                       IVwth=factor(rep(1:Q, each=P*Nj)),
                       DV=rnorm(Nj*P*Q, muJK, 3))
```



```r
summary(aov(DV ~ IVbtw*IVwth + Error(id/IVwth), data=dfSPFpqL))
```

```

Error: id
          Df Sum Sq Mean Sq F value  Pr(>F)    
IVbtw      2    178    89.2    14.9 4.3e-05 ***
Residuals 27    162     6.0                    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 

Error: id:IVwth
            Df Sum Sq Mean Sq F value  Pr(>F)    
IVwth        2    131    65.5    8.45 0.00064 ***
IVbtw:IVwth  4     43    10.9    1.40 0.24659    
Residuals   54    419     7.8                    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```


### Using `Anova()` from package `car` with data in wide format


```r
dfSPFpqW <- reshape(dfSPFpqL, v.names="DV", timevar="IVwth",
                    idvar=c("id", "IVbtw"), direction="wide")
```



```r
library(car)
fitSPFpq   <- lm(cbind(DV.1, DV.2, DV.3) ~ IVbtw, data=dfSPFpqW)
inSPFpq    <- data.frame(IVwth=gl(Q, 1))
AnovaSPFpq <- Anova(fitSPFpq, idata=inSPFpq, idesign=~IVwth)
summary(AnovaSPFpq, multivariate=FALSE, univariate=TRUE)
```

```

Univariate Type II Repeated-Measures ANOVA Assuming Sphericity

               SS num Df Error SS den Df     F  Pr(>F)    
(Intercept)  60.9      1      161     27 10.18 0.00359 ** 
IVbtw       178.4      2      161     27 14.92 4.3e-05 ***
IVwth       131.0      2      419     54  8.45 0.00064 ***
IVbtw:IVwth  43.4      4      419     54  1.40 0.24659    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 


Mauchly Tests for Sphericity

            Test statistic p-value
IVwth                0.981   0.779
IVbtw:IVwth          0.981   0.779


Greenhouse-Geisser and Huynh-Feldt Corrections
 for Departure from Sphericity

            GG eps Pr(>F[GG])    
IVwth        0.981     0.0007 ***
IVbtw:IVwth  0.981     0.2474    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 

            HF eps Pr(>F[HF])    
IVwth         1.06    0.00064 ***
IVbtw:IVwth   1.06    0.24659    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```


### Using `anova.mlm()` and `mauchly.test()` with data in wide format


```r
anova(fitSPFpq, M=~1, X=~0, idata=inSPFpq, test="Spherical")
```

```
Analysis of Variance Table


Contrasts orthogonal to
~0


Contrasts spanned by
~1

Greenhouse-Geisser epsilon: 1
Huynh-Feldt epsilon:        1

            Df    F num Df den Df  Pr(>F)  G-G Pr  H-F Pr
(Intercept)  1 10.2      1     27 0.00359 0.00359 0.00359
IVbtw        2 14.9      2     27 0.00004 0.00004 0.00004
Residuals   27                                           
```

```r
anova(fitSPFpq, M=~IVwth, X=~1, idata=inSPFpq, test="Spherical")
```

```
Analysis of Variance Table


Contrasts orthogonal to
~1


Contrasts spanned by
~IVwth

Greenhouse-Geisser epsilon: 0.9813
Huynh-Feldt epsilon:        1.0575

            Df    F num Df den Df Pr(>F) G-G Pr H-F Pr
(Intercept)  1 8.45      2     54 0.0006 0.0007 0.0006
IVbtw        2 1.40      4     54 0.2466 0.2474 0.2466
Residuals   27                                        
```



```r
mauchly.test(fitSPFpq, M=~IVwth, X=~1, idata=inSPFpq)
```

```

	Mauchly's test of sphericity
	Contrasts orthogonal to
	~1

	Contrasts spanned by
	~IVwth


data:  SSD matrix from lm(formula = cbind(DV.1, DV.2, DV.3) ~ IVbtw, data = dfSPFpqW) 
W = 0.981, p-value = 0.7789
```


Effect size estimates: generalized $\hat{\eta}_{g}^{2}$
-------------------------


```r
(anRes <- anova(lm(DV ~ IVbtw*IVwth*id, data=dfSPFpqL)))
```

```
Analysis of Variance Table

Response: DV
            Df Sum Sq Mean Sq F value Pr(>F)
IVbtw        2    178    89.2               
IVwth        2    131    65.5               
id          27    161     6.0               
IVbtw:IVwth  4     43    10.9               
IVwth:id    54    419     7.8               
Residuals    0      0                       
```



```r
SSEtot <- anRes["id", "Sum Sq"] + anRes["IVwth:id", "Sum Sq"]
SSbtw  <- anRes["IVbtw", "Sum Sq"]
SSwth  <- anRes["IVwth", "Sum Sq"]
SSI    <- anRes["IVbtw:IVwth", "Sum Sq"]
```



```r
(gEtaSqB <- SSbtw / (SSbtw + SSEtot))
```

```
[1] 0.2352
```

```r
(gEtaSqW <- SSwth / (SSwth + SSEtot))
```

```
[1] 0.1842
```

```r
(gEtaSqI <- SSI   / (SSI   + SSEtot))
```

```
[1] 0.0696
```


Or from function `ezANOVA()` from package [`ez`](http://cran.r-project.org/package=ez)

Simple effects
-------------------------

Separate error terms

### Between-subjects effect at a fixed level of the within-subjects factor


```r
summary(aov(DV ~ IVbtw, data=dfSPFpqL, subset=(IVwth==1)))
```

```
            Df Sum Sq Mean Sq F value Pr(>F)  
IVbtw        2   81.8    40.9    4.69  0.018 *
Residuals   27  235.3     8.7                 
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```

```r
summary(aov(DV ~ IVbtw, data=dfSPFpqL, subset=(IVwth==2)))
```

```
            Df Sum Sq Mean Sq F value Pr(>F)  
IVbtw        2   37.2    18.6    2.81  0.078 .
Residuals   27  178.2     6.6                 
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```

```r
summary(aov(DV ~ IVbtw, data=dfSPFpqL, subset=(IVwth==3)))
```

```
            Df Sum Sq Mean Sq F value Pr(>F)   
IVbtw        2    103    51.4    8.33 0.0015 **
Residuals   27    167     6.2                  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```


### Within-subjects effect at a fixed level of the between-subjects factor


```r
summary(aov(DV ~ IVwth + Error(id/IVwth), data=dfSPFpqL,
        subset=(IVbtw=="A")))
```

```

Error: id
          Df Sum Sq Mean Sq F value Pr(>F)
Residuals  9   22.6    2.51               

Error: id:IVwth
          Df Sum Sq Mean Sq F value Pr(>F)  
IVwth      2     47   23.49    3.51  0.052 .
Residuals 18    120    6.69                 
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```

```r
summary(aov(DV ~ IVwth + Error(id/IVwth), data=dfSPFpqL,
        subset=(IVbtw=="B")))
```

```

Error: id
          Df Sum Sq Mean Sq F value Pr(>F)
Residuals  9   23.1    2.57               

Error: id:IVwth
          Df Sum Sq Mean Sq F value Pr(>F)   
IVwth      2    111    55.7    8.15  0.003 **
Residuals 18    123     6.8                  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
```

```r
summary(aov(DV ~ IVwth + Error(id/IVwth), data=dfSPFpqL,
        subset=(IVbtw=="C")))
```

```

Error: id
          Df Sum Sq Mean Sq F value Pr(>F)
Residuals  9    116    12.9               

Error: id:IVwth
          Df Sum Sq Mean Sq F value Pr(>F)
IVwth      2     16    8.00    0.82   0.46
Residuals 18    175    9.74               
```


Planned comparisons for the between-subjects factor
-------------------------


```r
mDf    <- aggregate(DV ~ id + IVbtw, data=dfSPFpqL, FUN=mean)
aovRes <- aov(DV ~ IVbtw, data=mDf)
```

```r
cMat <- rbind("-0.5*(A+B)+C"=c(-1/2, -1/2, 1),
                       "A-C"=c(-1,    0,   1))
```



```r
library(multcomp)
summary(glht(aovRes, linfct=mcp(IVbtw=cMat), alternative="greater"),
        test=adjusted("none"))
```

```

	 Simultaneous Tests for General Linear Hypotheses

Multiple Comparisons of Means: User-defined Contrasts


Fit: aov(formula = DV ~ IVbtw, data = mDf)

Linear Hypotheses:
                  Estimate Std. Error t value Pr(>t)
-0.5*(A+B)+C <= 0   -2.375      0.547   -4.34      1
A-C <= 0            -3.421      0.631   -5.42      1
(Adjusted p values reported -- none method)
```


Detach (automatically) loaded packages (if possible)
-------------------------


```r
try(detach(package:multcomp))
try(detach(package:mvtnorm))
try(detach(package:car))
try(detach(package:survival))
try(detach(package:splines))
try(detach(package:nnet))
try(detach(package:MASS))
```


Get the article source from GitHub
----------------------------------------------

[R markdown](https://github.com/dwoll/RExRepos/raw/master/Rmd/anovaSPFpq.Rmd) - [markdown](https://github.com/dwoll/RExRepos/raw/master/md/anovaSPFpq.md) - [R code](https://github.com/dwoll/RExRepos/raw/master/R/anovaSPFpq.R) - [all posts](https://github.com/dwoll/RExRepos/)
