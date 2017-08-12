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
* $TS_i < 10$ or ${\Delta}F_i / F_i > 0.5$ (or $Npred_i < 3$)
* $95\%$ Bayesian UL when $TS_i < 1$
* $95\%$ profile method otherwise. [$\delta = chi2inv(2*(0.95-0.5))/2 = 2.71/2$][^chi2inv]
5. Divide data into bins (gtselect).
6. Run likelihood analysis on each.
7. Check each analysis for problems.
8. Compute upper limits where necessary.

***
[^chi2inv]: chi2inv (x, n): For each element of x, compute the quantile (the inverse of the CDF) at x of the chi-square distribution with n degrees of freedom.