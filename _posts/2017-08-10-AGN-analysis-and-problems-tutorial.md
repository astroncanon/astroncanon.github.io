---
layout: post
title: AGN analysis and problems tutorial
categories: [FermiData]
---

> by *Stephen Fegan*

## Overview
* AGN analysis is much the same as for other source types, but some tasks more relevant, so certain "problems" more common
* Light curve calculation - tutorial / problems
* Variability testing - 1FGL and 2FGL methods
* Absolute "goodness 0f fit" measure in spectral modeling
* Flux / Index correlations

## Light curve tutorial
* LC creation: Regular / adaptive binning with LA, AP method, and Bayesian blocks (constant rate segments)
* Choice depends on Ur needs
* Flux v.s. Flux/Index LC

## Regular flux calculation with likelihood
1. Perform a standard LA of full time period.
2. Determine what binning is reasonable. 
* Science goals and strength of source of interest.
* Bins should not be much larger than $TS_{Tot} / 25$.
* Affection of upper limits on the ayalysis.
* Avoid periods close to being integer fractions of the orbital precessional period of $53.7$ days.
3. Prepare an ROI model for the time bins. 
* Freeze spectral shapes of all background sources.
* Freeze all parameters of weak background sources - Will cause convergence problems. $TS_{Tot} / N_{bin} < 4$ or $9$.
* For flux only LC, freeze spectral shape of source of interest.
4. Decide on criteria for upper limits. 
* $TS_i < 10$ or ${\Delta}F_i / F_i > 0.5$ (or $N_{pred}_i < 3$)
* $95\%$ Bayesian UL when $TS_i < 1$
* $95\%$ profile method otherwise. [$\delta = chi2inv(2*(0.95-0.5))/2 = 2.71/2$][^chi2inv]
5. Divide data into bins (gtselect).
6. Run likelihood analysis on each.
7. Check each analysis for problems.
8. Compute upper limits where necessary.

## Example from 2FGL
![PKS_0528+134]({{ site.url }}/images/posts/2017-08-10-PKS-0528+134.png)

## Error-matrix problems in LCs
* The minimizer may not give proper error matrix in calculation of LC points.
* Usually the minimizer will converge, and the model parameters will be OK.
* But the errors can be VERY wrong.
* Hence a $chi^2$ fit to a constant can be wrong.
* This seems to be related to parameters that are not properly constrained (and hence hit limits).

## A sample LAT LC
![sample_LC]({{ site.url }}/images/posts/2017-08-10-sample-LC.png)

* In this plot, right point has smaller error bar than others at same flux level.
* Left point has error bar so small it is not visible. Potential disaster for variability test.

## Automated test?
* Need some automatic way to find points whose errors are "too small"
* Minimum error (variance) is given by Cramer-Rao bound. But that is not calculated in ST.
* Recognize that if source model has converged and predicts $N_{pred}$ counts, then ratio of flux error to flux ($\sigma_F/F$) should not be better than that of Poisson counts underlying measurement: $\sqrt{N_{pred}}/N_{pred}$.

## Simple test to find such points
![simple tesst]({{ site.url }}/images/posts/2017-08-10-simple-test.png)

* Plot of $F / F_{err}$ to $N_{pred} / \sqrt{N_{pred}}$ and find outliers. These should be investigated.

```python
from math import *
import BinnedAnalysis
obs = ...
like = ...
like.fit()
src = 'VER0521'
flux = like.normPar(src).getValue()
error = like.normPar(src).error()
N_{pred} = like.N_{pred}Value(src)
print error/flux, sqrt(N_{pred})/N_{pred}
# 0.0281931953422 0.421340108935
print like.optObject.getRetCode()
# 102 --> Anything other than zero indicates problems
```
## Check MINUIT status
`gtlike chtter=3 ...`:

```python
ERR MATRIX NOT POS-DEF
Minuit fit quality: 2 estimated distance: ...
```
`pyLikelihood`:

```python
obs = BinnedObs(...)
like = BinnedAnalysis(obs, 'model.xml', 'MINUIT')
minuit_obj = pyLike.Minuit(like.logLike)
like.fit(covar=True, optObject=minuit_obj)
distance_from_minimum = minuit_obj.getDistance()
qual = minuit_obj.getQuality()
```
$0$: Error matrix not calculated at all  
$1$: Diagonal approximation only, not accurate   
$2$: Full matrix, but forced positive-definite (i.e. not accurate)   
$3$: Full accurate covariance matrix (After MIGRAD, this is the indication of normal convergence.)

## Conjunctions with the Sun
Conjunctions with the Sun can lead to contamination of the LC by gamma rays from the Sun. In 2FGL we flag all periods when the sun-to-source separation is less than 2.5deg.

* Always check the ecliptic coordinates of the source.
* For sources with low ecliptic latitude remove time
periods where the source-to-Sun separation is small.

## Recommendations for LC
* Follow the strategy of 1FGL / 2FGL.
* Freeze spectral shape parameters ($\Gamma$, $\alpha$/$\beta$, $E_b$, etc).
* Freeze very weak background sources that have very low TS values.
* Set threshold on $TS$, ${\Delta}F/F$ (and $N_{pred}$?) and calculate upper limits for weaker sources.
* Look for suspicious points! Check MINUIT status.

## Variability testing
* If you claim variability (or lack of it) then a quantitate test is a good idea.
* Different variability tests are available and may
be useful in certain circumstances.  
\- 1FGL $\chi^2$ test for a constant flux  
\- 2FGL likelihood test for a constant flux  
\- Bayesian blocks test for Poisson process
* 2FGL method is more sensitive than 1FGL (but
more complex to compute).

## 1FGL variability index
$\chi^2$ criterion based on best-fit fluxes and flux errors in LC.

$$
\begin{equation}
   w_{i} = \frac{1}{\sigma_{i}^{2} + (f_{rel}F_{i})^{2}}  \\
   F_{wt} = \frac{\sum_{i}w_{i}F_{i}}{\sum_{i}w_{i}} \\
   V = \sum_{i}w_{i}(F_{i} - F_{wt})^{2}
\end{equation}
$$

Where $f_{rel}$ is the systematic errors.  
No variability: $V \sim \chi^2(N-1)$.  
Prescription to include upper limits and systematic errors in exposure.

***
[^chi2inv]: chi2inv (x, n): For each element of x, compute the quantile (the inverse of the CDF) at x of the chi-square distribution with n degrees of freedom.