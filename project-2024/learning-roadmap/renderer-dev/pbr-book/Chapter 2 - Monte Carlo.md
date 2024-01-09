
# Introduction
---

Rendering is full of integration problems.
Integral equations should be turned into numerical methods.

It is basically sampling randomly to evaluate integrals with a convergence rate that is independent of the dimensionality of the integrand.

Randomized algorithms fall broadly into two classes
- *Las Vegas algorithms*
- *Monte Carlo algorihtms*

# Monte Carlo : Basics
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

If a sampe $X_i=(x_i,y_i,z_i)$ are chosen uniformly from the cube range '$[x_0, x_1]\times[y_0, y_1]\times[z_0, z_1]$'
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