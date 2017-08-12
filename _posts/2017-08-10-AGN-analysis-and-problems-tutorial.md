# AGN analysis and problems tutorial
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
* Science goals and strength of source of interest
* Bins should not be much larger than TS[Tot]/25
* `Affection of upper limits on the ayalysis`
* `Avoid periods close to being integer fractions of the orbital precessional period of 53.7 days`
4. Prepare an ROI model for the time bins. </br>`Freeze spectral shapes of all background sources`</br>`Freeze all parameters of weak background sources - Will cause convergence problems. TS[Tol]/bins < 4 or 9`</br>`For flux only LC, freeze spectral shape of source of interest.`
5. Decide on criteria for upper limits. </br>`TS[i] < 10 or Ferr[i]/F[i] > 0.5 (or Npred[i] < 3)`
6. 