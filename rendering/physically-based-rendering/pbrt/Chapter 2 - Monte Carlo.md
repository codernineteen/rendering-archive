
# Introduction
---

Rendering is full of integration problems.
Integral equations should be turned into numerical methods.

It is basically sampling randomly to evaluate integrals with a convergence rate that is independent of the dimensionality of the integrand.

Randomized algorithms fall broadly into two classes
- *Las Vegas algorithms*
- *Monte Carlo algorihtms*

# Monte Carlo Integration : Basics
---
It is based on randomization.
randomization is related to probability and statistics as its foundation.

## 2.1.1 Probability Review
---
Random variables are always drawn from some domain, which can be either discrete or continuous.

When a Random variable is X, a new random Variable Y denotes :
$$Y=f(X)$$
by applying a function.

For example, There is a random variable X
$$X_i\in\set{1,2,3,4,5,6}$$
Each event has 1/6 probability and the sum of probabilities always equal to 1.
We call this case that has same probability for all potential values as *uniform* .

- PMF(probability mass function) : a function p(X) that gives a discrete random variable's probability.
	In above case, $p(X)=\frac{1}{6}$

- joint probability $p(X,Y)=p(X)p(Y)$ if X and Y are independent.
- In dependent case, $p(X,Y)=p(X)p(X|Y)=p(X)\frac{p(X\cap Y)}{p(Y)}$ 
	we call this 'conditional probability'.
	For example, pick a white ball after previous picking when the ball in bucket is finite.
	Another example is that choice of light conditioned on 'surface normal' and 'receiving point'.

- Canonical uniform random variable($\xi$) : all value in its domain 0<= x < 1 independently with **uniform** probability.
	- ex) probability of sampling illumination from each ligth based on its power relative to the total power from all sources.$$p_i=\frac{\Phi_i}{\sum_{j}\Phi_j}$$
- cumulative distribution function : probability that a value less than or equal to some value x
- continuous random variable : values over ranges of continuous domain
- **PDF(probability density function)** :
	the relative probability of a random variable taking on a particular value and is the continuous analog of the PMF. $$p(x)=\frac{dP(x)}{dx}$$, where P(x) is integration of CDF
	Given an interval $[a,b]$ in the domain, $$Pr\set{x\in[a,b]}=\int_a^bp(x)dx=P(b)-P(a)$$

- Expected value of function : a sum of certain function value multiplied by its probability in domain. $$E_p[f(x)]=\int_Df(x)p(x)dx$$


## 2.1.3 Monte Carlo Estimator
---
Monte carlo estimator approximates the value of an arbitrary integral.

Given a supply of independent uniform variables $X_i\in[a,b]$,
$E[F_n]$ is equal to the integral.
- Derivation : Because sum of p(x) is equal to 1
- $$E[F_n]=E[\frac{b-a}{n}\sum_{i=1}^{n}f(X_i)]=\frac{b-a}{n}\sum_{i=1}^{n}E[f(X_i)]= \frac{b-a}{n}\sum_{i=1}^{n}\int_a^bf(x)p(x)dx=\frac{1}{n}\sum_{i=1}^{n}\int_a^bf(x)dx=\int_a^bf(x)dx$$

When n independent samples X_i are taken from uniform multidimensional PDF, consider 3d this integral
![](../../../../images/Pasted%20image%2020240109090155.png)

If a sampe $X_i=(x_i,y_i,z_i)$ are **chosen uniformly** from the cube range '$[x_0, x_1]\times[y_0, y_1]\times[z_0, z_1]$'
- The PDF p(X) 
	![](../../../../images/Pasted%20image%2020240109090435.png)
- the monte carlo estimator![](../../../../images/Pasted%20image%2020240109090454.png)


By generalization, we can relax restriction to uniform random variables.
	-> reducing error
Below equation can be used instead of integral
![](../../../../images/Pasted%20image%2020240109090702.png),
where the random variable X_i are drawn from a PDF $p(x)$
Note that p(x) must not be zero in this case.

- derivation
![](../../../../images/Pasted%20image%2020240109090848.png)

## 2.1.4 Error in Monte Carlo estimators

Variance in Monte carlo estimator is important to justify its use.
a few of properties of variance below:
![](../../../../images/Pasted%20image%2020240111164716.png)
![](../../../../images/Pasted%20image%2020240111164902.png)
![](../../../../images/Pasted%20image%2020240111164908.png)

If the estimator is a sum of independent random variables
-> the variance of sum is the sum of individual random variables' variances.
![](../../../../images/Pasted%20image%2020240111165040.png)
This equation shows us that variance decreases linearly with the number of samples n.
The error in a Monte Carlo estimate only goes down at a rate of $O(n^{-\frac{1}{2}})$

**Monte carlo is the only practical numerical integration algorithm for high-dimensional integrations.**

Linear decrease in variance with increasing number of samples
-> easy to compare two different estimators.

ex )
- first estimator's variance : x
- second estimator's variance : x/2, but three times as long to compute an estimate.
In this case, first one is preferable.
It can take 3 times samples more in time consumed by second estimator -> 3 x variance reduction.

**Efficiency of an estimator**
$$\epsilon[F]=\frac{1}{V[F]T[F]}$$
Where, V is variance and T is runtime to compute.

**Additionall, Not all estimators of integrals have expected value that is equal to the value of the integral** -> said to be biased
The amount of bias is :
$$\beta=E[F]-\int f(x)dx$$
Biased estimators may be desirable if they are able to get close to the correct result more quickly than 'unbiased' one.
Ex)
- $\frac{1}{n}\sum_{i=1}^{n}X_i$  -> unbiased, has variance with $O(n^{-1})$ 
- $\frac{1}{2}max(X_1, X_2, ..., X_n)$ 
	- expected value is $0.5\frac{n}{n+1}\ne0.5$ -> biased 
	- variance is $O(n^{-2})$  -> **better**


 ## Mean Square Error
 - F : true value
 - integral of f(x) : value from estimator.
 ![](../../../../images/Pasted%20image%2020240111174616.png)
 - for unbiased estimator, MSE is equal to the variance
 - otherwise, sum of variance and the squared bias of the estimator.

If an accurate estimate of the integral $\tilde F \approx \int f(x)dx$  can be computed, then the mean squared error can be estimated by :
![](../../../../images/Pasted%20image%2020240114174320%201.png)


# Monte Carlo Integration : Improving Efficiency
---
From unbiased Monto Carlo estimator, 
reliable relationship : number of samples <-> Variance(and thus, error)

We can improve noisy rendered image by increasing the number of samples to reduce error.

**The only option for variance reduction is to find ways to make more of the samples that can be taken.**


## 1. stratified sampling
A classic and effective techniqued for variance reduction is based on the placement of samples in order to **be less likely to miss important features.**

- a stratified sampling
	'decomposes' integration domain into regions and 'places' samples in each regions.
	![](../../../../images/Pasted%20image%2020240115222835%201.png)
	,where union of each regions cover the original domain.
	

For example, supersampling a pixel.
A pixel is divided into a $k \times k$ grid.
a sample is drawn uniformly within each grid cell. (this prevents the sample locations from being clumped)

Within a single stratum, the monte carlo estimate is
![](../../../../images/Pasted%20image%2020240115223632%201.png)
Where $X_{i,j}$ is the jth sample drawn from density $p_i$.

The overal estimate is $F=\sum_iv_iF_i$ , where $v_i$ is the fractional volume of stratum i. 

- True value of the integrand in stratum i 
![](../../../../images/Pasted%20image%2020240115223917%201.png)
- the variance in this stratum is 
![](../../../../images/Pasted%20image%2020240115224021%201.png)

Hence, the variance of the per-stratum estimator $\sigma^2/n_i$ with $n_i$ samples in the stratum.
![](../../../../images/Pasted%20image%2020240115224453%201.png)
If we assume $n_i$ is proportional to the volume $v_i$, then we have $n_i=v_in$ and above variance will be
![](../../../../images/Pasted%20image%2020240115224659%201.png)

To compare this variance with a method without stratification,
Choosing an unstratified sample ==
Choosing a random stratum I according to the discrete probability distribution( defined by)
+
choosing a random sample X in the stratum I. 
-> trivially choosing X is conditional propability
![](../../../../images/Pasted%20image%2020240115225602%201.png)
where $Q$ is the mean of $f$ over the whole integrand domain.

Two things to notice about above equation :
- right-hand sum should be non-negative
- stratification always reduces variance unless right-hand sum is exactly 0. (never increase variance.)

For the best output, we want to increast right-hand sum.

Downside of stratified sampling is 'curse of dimensionality'.
-> Full stratification in D dimensions with S strata per dimension required $S^D$ samples.
Fortunately, it is often possible to stratify some of the dimensions independently and then randomly associate samples from different dimensions.

## 2. Importance Sampling
---
Monte carlo estimator converges more quickly **if the samples are taken from a distribution $p(x)$ that is similar to the function $f(x)$ in the integrand**
-> importance sampling utilize this feature to reduce variance.

Let's see why the distributions reduce error.
When $p(x)$ is proportional to $f(x)$ ( $p(x) = cf(x)$ ), Normalization of the PDF requires :
![](../../../../images/Pasted%20image%2020240116111956%201.png)
-> p(x) = a probability with which f(x) comes out from integration of f(x). 

Although this requires that we know the value of the integral, if we could sample from this distribution, each term of the sum in the estimator would have the value
![](../../../../images/Pasted%20image%2020240116112545%201.png)

It is unreasonable because we would not use monte carlo method if we could integrate f directly.
-> But if a density $p(x)$ can be found that is similar in shape to $f(x)$, variance reduced.

Note that importance samplling can increase variance if a poorly chosen distribution is used.

**In conclusion, importance sampling is to sample from another probability distribution that makes sampling easier when we have a difficulty in sampling with original probability distribution. Also we need to care the 'arbitrary distribution' should represent original one in a good way to reduce variance.**


## 3. Multiple importance sampling
---
Integrals that are the product of two or more functions :
ex. $\int f_a(x)f_b(x)dx$
It is often possible to derive separate importance sampling strategies for individual factors.
This situation is common in the integrals involved with 'light transport'. (product of BSDF, incident radiance and a cosine factor in the light tranport).

Let's assume we have two sampling distributions $p_a$ and $p_b$ that match distributions of $f_a$ and $f_b$ exactly.
Following monte carlo equation,
- draw samples using $p_a$ and the estimator will be
![](../../../../images/Pasted%20image%2020240116115213%201.png)
The variance of this estimator is proportional to the variance of $f_b$.
In the opposite case that we draw samples using $p_b$, this fact is kept true.

Multiple importance sampling (MIS) comes in handy in this case.
The idea is to draw samples from multiple sampling distributions when estimating an integral.
**MIS provides a method to weight the samples from each technique that can eliminate larage variance spikes due to mismatches between the integrand's value and the sampling desnity.**

With two sampling distributions $p_a$ and $p_b$ and a single sample taken from each one , MIS monte carlo estimator is
![](../../../../images/Pasted%20image%2020240116115932%201.png)
where $w_a$ and $w_b$ are weighting functions chosen that the expected value of this estimator is the value of the integral of $f(x)$.

With generalizing this estimator,
![](../../../../images/Pasted%20image%2020240116195657.png)

In practice , a good choice for the weighting functions is given by the 'balance heurisitc'
![](../../../../images/Pasted%20image%2020240116195906.png)
And the power heuristic often reduces variance even further.
![](../../../../images/Pasted%20image%2020240116200647.png)

# Reference
https://www.pbr-book.org/4ed