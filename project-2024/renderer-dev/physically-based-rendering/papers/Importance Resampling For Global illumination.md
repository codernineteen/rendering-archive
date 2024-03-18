

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

# 2. Sample generation technique.

# 2.1 CDF inversion

- the most common sampling technique in global illumination
- permits exact sampling from the desired distribution
	- a CDF is the integral of the pdf : $F(x) = \int_{\infty}^{x}p(x')dx'$
	- a CDF  is uniformly distributed between 0 and 1
		$\xi = F(x)$ , where $\xi$ means uniform distribution
	- Then if we inverse the equation ,
		$x = F^{-1}(\xi)$
the inversion method should satisfy
- normalized pdf
- a closed form integral
- an invertible CDF



# 2.2 Rejection sampling

For this method we need two density that satisfies
- $\hat{q}(\theta)$ should be evaluated (this is the density that we want to draw a sample from)
- we should have a pdf $\hat{p}(\theta)$ that bounds $q$.

if $\frac{\hat{q}}{\hat{p}M} > \xi$ , we accept the sample.

To be efficient $\hat{p}$ should tightly bound to $\hat{q}$


# 2.3 sampling importance resampling

- a common method in computational statistics for generating samples from difficult distributions.
	- for example, sequential importance sampling, particle filtering

**Importance resampling** can generate samples that approximately have the distribution $\frac{\hat{q}}{C}=q$.
1. generate a set of 'proposal' M samples from a source distribution $p$
2. weight these samples appropriately.
3. resample these  M samples by drawing samples from them with probability proportional to their weights. (new N resampled samples)

For example, if we choose $w(x_j) = \frac{\hat{q}(x_j)}{p(x_j)}$ , resulting samples will be approximately distributed according to $\hat{q}$.

The more number of M increases, the more the distribution of each sample approaches $q$.
![](../../../../Pasted%20image%2020240314115659.png)

Importance resampling usage in global illumination literatures.
- decrease number of visibility tests necessary in bidirectional path tracing
- improve direct lighting computations
- sample the distribution of the product of a Phong BRDF model and an illuminating environment map.

## 2.4 Metropolis sampling

- generates a Markov chain with a stationary distribution that is equal to the desired sampling distribution $\hat{q}$.

- Markov chain is formed by proposing a new sample $x'$ from a distribution called the transition function $T$ (conditioned on the current sample $x$)

- the proposal sample becomes current sample if 
	$$\xi < min(1, \frac{\hat{q}(x')T(x|x')}{\hat{q}(x)T(x'|x)})$$
	otherwise, the old sample is retained as the current sample.

One of good things of Metropolis sample is that we often do not need perfect sampling for global illumination because the monte carlo integration with appropriate weighting will be unbiased.


## 2.5 Gibbs sampling

- special case of Metropolis (in other words, Metropolis agorithm is generalization of special cases.)
- details of this algorithm is not handled in this article.

---

# 3. Monte Carlo integration

- probabilistic method to integrate difficult functions.
- Due to its generality, it is commonly used to solve problems in high dimensions and discontinuous functions.
- subject to variance because of probabilistic nature.
	variance is noisy. 
	Many methods have been invented to reduce this variance in monte carlo integration.

## 3.1 basics of monte carlo integration
- a monte carlo estimator can be shown as:
$$\int_{\Omega}f(w)dw \approx\frac{1}{N}\sum_{i=1}^{N}\frac{f(x_i)}{q(x_i)}$$
, where pdf $q$ is normalized sampling distribution.

## 3.2 variance reduction

The variance of the Monte carlo integration estimator is 
$$V(\sum_{i=1}^{N}\frac{f(x_i)}{q(x_i)})=\frac{1}{N}V(\frac{f(w)}{q(w)})$$.
This variance expression implies that the error of the estimate only decreses with $O(N^{1/2})$ (standard deviation : $\sqrt{\frac{1}{N}(...)}$, so the error decreases proportional to $\sqrt{N}$)
-> this convergence rate is quite slow -> we need methods to reduce variance.

Here are some of variance reduction techniques.

1. Importance sampling
	The variance of the estimator is proportional to $f(w)/q(w)$. If we can reduce this ratio, the overall variance will be reduced.
	**Importance sampling** is a method to choose a proper $q(w)$ to minimize the variance of the ratio. (ideally $q \propto f$, that is $q = cf$ where c is a constant)
	
	- Practically, we choose $q$ that mimics $f$.
		- the distribution must be normalized 
		- it must be easy to generate samples from the distribution.
		-> we choose a simple function that has a known sampling method in general.
		
	If a specific integrand is going to be integrated repeatedly, we can take two steps to find a good and flexible $q$.
	- q must be flexible enough to match $f$ while it remains normalized and easy to sample
	- a method for precomputing the functional parameters must be derived.
	  
	Examples in global illumination)
	- structured importance sampling
	- a factored BRDF representation
	- Wavelet importance sampling

2. Defensive Importance sampling
	If we choose a poor $q$, importance sampling still can increase variance.
	For example,
	- $q$ is so small, but $f$ is relatively very high -> possibly reaches out infinity
	- $f$ is so small, but q is relatively very high -> can be close to zero.
	To prevent first case, we can replace $q$ with mixture density that includes a uniform density.$$q'(w)= \alpha q(w) + (1-\alpha)u(w)$$
		, where $u(w)$ is an uniform distribution.

3. Multiple importance sampling.
	The MIS method is a generalized version of defensive importance sampling
	$$q'(w)=\sum_{i=1}^{K}\alpha_i q_i(w)$$
	 we also can use a weighted Monte Carlo estimator.
	 $$\int_{\Omega}f(w)dw\approx\frac{1}{N}\sum_{i=1}^{N}w(x_i)\frac{f(x_i)}{q(x_i)}$$
4. Weighted Importance Sampling
	Modified Monte carlo estimate to use a second PDF g.
	$$\int_{\Omega}f(w)dw \approx \frac{\frac{1}{N}\sum_{i=1}^{N}\frac{f(x_i)}{q(x_i)}}{\frac{1}{N}\sum_{i=1}^{N}\frac{g(x_i)}{q(x_i)}}$$
	If $g$ mimics $f$ well, the numerator and the denominator will flutuate jointly and the variance of the ratio wil be reduced. (ideally, $g$ must mimic $f$ better than $q$)
5. Stratified sampling
6. Correlated Sampling
	$$\int_{\Omega}f(w)dw \approx \frac{1}{N}\sum_{i=1}^{N}\frac{f(x_i)-g(x_i)}{q(x_i)}+\int_{\Omega}g(w)dw$$
	This method works when a $g$ approximates $f$ and ($f$-$g$) is nearly constant.


# 4. Resampled Importance sampling

- robust variance reduction technique.
- no prior required. 
	-> this property of RIS makes it more robust than standard importance sampling that required priori knowledge of the integrand.

The integral $I$ of a function $f(x)$ :
$$I = \int_{\Omega}f(x)d\mu(x)$$
If we use Monte carlo integration :
$$I \approx \hat{I}_{MonteCarlo}=\frac{1}{N}\sum_{i=1}^{N}\frac{f(y_i)}{\hat{q}(y_i)}$$, where $\hat{q}$ is normalized and easy to sample using rejection or inversion sampling.

Because of the constraints of standard Monte carlo estimator, we need to modify it.

**Quick review for SIR (Sampling Importance Resampling)**
	When we work with Importance sampling, we use a little trick by multiplying $\frac{q(x)}{q(x)}$, where $q$ is a easy distribution to sample.
	Then, the target function that we want to integrate is converted into :
		$$F(x) = \frac{f(x)}{q(x)}q(x)$$
	The basic idea of SIR is to use $f(x)/q(x)$ as weight.
	For example, $w_i = \frac{f(x_i)}{q(x_i)}$, where the sample $x_i$ is drawn by the distribution of $q$.
	And for all i, we normalize the specific weight of i by their summation.
	$$g_i = \frac{w_i}{\sum w_j}$$
	By using these new probability, we resample new samples from the samples already taken by $q$ with new distribution of $g$.

The basic steps of RIS
1. To generate samples from $\hat{q}$, use SIR
2. The samples generated by SIR are only approximately distributed according to $\hat{q}$. 
	we must add a weighting term to the standard Monte Carlo estimator to maintain unbiasedness of it.
	For generality, allow the weighting term to be a function of both the proposals $\{{x_1, ...x_M}\}$ and the resulting samples $\{y_1, ... y_N\}$}.
		$$I_{ris}=\frac{1}{N}\sum_{i=1}^{N}\frac{f(y_i)}{\hat{q}(y_i)}\cdot w^*(x_1, ..., x_M,y_1,...,y_N)$$
		,where $\hat{q}$ is unnormalized and the density of the samples $y_i$ only approximates $\hat{q}$.
		
	The weight function is simply the average of sum of weights
	$$w^*(x_1, ..., x_M,y_1,...,y_N)=\frac{1}{M}\sum_{j=1}^{M}w(x_j)$$
	By combining two equations together, we get a basic RIS estimator :
		$$I_{ris}=\frac{1}{N}\sum_{i=1}^{N}\frac{f(y_i)}{\hat{q}(y_i)}\cdot\frac{1}{M}\sum_{j=1}^{M}w(x_j)$$ 
For RIS to be unbiased , Below two conditions should be satisfied.
- sampling density $\hat{q}$ and proposal density $p$ must be greater than zero everywhere that $f$ is non-zero.
- $M,N$ must be greater than zero.