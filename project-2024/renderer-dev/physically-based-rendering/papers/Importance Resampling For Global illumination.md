

# 1. Introduction

Global illumination : the process of turning a virtual scene into a realistic image by simulated light transport.
-> this is difficult to solve using standard-numerical techniques
-> therefore, we should approximate the real root by using analytical techniques like Monto Carlo Integration.
-> But monte carlo integration generates unacceptable noises(variance) in the scene because of its randomness nature.
-> The goal is to reduce this variance without compromising the generality of Monte Carlo integration.

Why we need the robust variance reduction techniques ?
1. the global illumination(shorty 'gi' from now on) problem is high-dimensional and the shape of integrand is complex and discontinuous.
2. we want to find algorithms that can be applied to various problems without modifications.

The 'Resampled Importance Sampling' method results in lower variance estimates for the types of problems that are commonly encontered in GI.

## 1.1 Global illumination
- an application of light transport
- light transport is to compute how light propagates from light source through a scene consiting of reflectors and absorbers.
- To simulate this light transport , rendering equation was invented.
$$L_o(x,w_o,\lambda, t)=L_e(x,w_o,\lambda, t)+L_r(x,w_o,\lambda, t)$$
$$L_r(x, w_o, \lambda,t)=\int_{\Omega}f_r(x,w_i,w_o,\lambda, t)L_i(x,w_i,\lambda, t)(w_i\cdot \text{n})dw_i$$

- integrating this equation is very difficult
	-> It is the Monte carlo integration that the only one integration approach that is capable of solving the completely general rendering equation.

But the monte carlo method is a probabilistic approach , so it leads to variance problem.

This noise can be reduced with
- more samples 
	this reduces noise in a scene effectively, but also simply. But it increases overhead to converge the intergral.
- variance reduction technique. (statistical tricks)


## 1.2 summary of original contributions

### Resampled Importance Sampling
- generate **samples** from a proposal distribution to estimate integral.
- then the samples are filtered using a resampling step.
- the result samples after resampling are used in a modified Monte carlo estimator.

Here are some of improvement using RIS :
- unbiased
- true generalization of standard importance sampling
- no precomputation
- doesn't rely on any specific properties of the function being integrated
- sampling density doesn't need to be normalized.