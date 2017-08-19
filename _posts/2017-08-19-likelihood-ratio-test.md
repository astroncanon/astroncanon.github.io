---
layout: post
title: Likelihood Ratio Tests
categories: Math
---

Likelihood ratio tests (LRTs) have been used to compare two **nested** models. The form of the test if suggested by its name,

$$
\begin{equation}
LRT = -2 log_{e}[\frac{L_{s}(\theta)}{L_{g}(\theta)}]
\end{equation}
$$

the ratio of two likelihood functions; the simpler model (s) has fewer parameters than the general (g) model. Asymptotically, the test statistic is distributed as a $\chi^2$ random variable, with degrees of freedom equal to the difference in the number of parameters between the two models.

Likelihood ratio tests compare two models provided the simpler model is a special case of the more complex model (i.e., “nested"). LRTs can be presented as a difference in the loglikelihoods (recall that $log(A/B) = logA – logB)$ and this is often handy as they can be expressed in terms of deviance. Then,

$$
\begin{equation}
LRT = -2 [log_{e}(L_{s}) - log_{e}(L_{g})]\\  
    = -2 log_{e}(L_{s}) + log_{e}(L_{g})\\  
    = deviance_{s} - deviance_{g}  
\end{equation}
$$

Thus, the LRT can be computed as a difference in the deviance for the two models (ignoring the term for the saturated model). This is convenient as the deviance is a value of interest in other respects.

Say we flipped two coins, we could have

$$
\begin{equation}
H_{o}: p_1 = P_2 \equiv p ~~~ ({\rm the~simpler~model,~1~parameter})
\end{equation}
$$

vs. the alternative of different probabilities of heads, hence

$$
\begin{equation}
H_{a}: p_1 \neq P_2 ~~~ ({\rm the~model~general~model,~2~parameters})
\end{equation}
$$

Compute MLEs under both models and compute the deviances ($D$),

$$
\begin{equation}
D_{s} = -2log_{e}(L(p)), ~~~ K_{s} = 1, {\rm the~number~of~parameters}\\
D_{g} = -2log_{e}(L(p_1,p_2)), ~~~ K_{g} = 2, {\rm the~number~of~parameters}
\end{equation}
$$

The the $LRT = D_s - D_g, df = K_g - K_s = 1$.

Thus, this test statistic in approximately $\chi^2$ with 1 df under the null hypothesis. The approximation improves as sample size increases. Note, too that the log-likelihood for the saturated model is a constant and the same for both of the above models; thus it was deleted in this example.

Testing of null hypotheses has seen decreasing use in many areas of applied science over the past 2 decades. We will made some reference to LRTs so that students can better understand existing literature that makes use of these methods. Problems with statistical hypothesis testing will be outlined at a latter point.