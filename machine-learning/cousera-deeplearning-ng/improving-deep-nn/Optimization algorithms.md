Training large data set is slow -> we need optimization algorithms to make it faster.

# 1. Mini-batch
---
With a general vectorization, when we have these training samples : $$X = [x^{(1)}, .. , x^{(n)}]$$, where n =  5,000,000
We need to wait a huge amount of time before gradient descents computation until all of the computation of cost function are finished.

What if we can separate this single large data set into several tiny training samples?
we may can start gradient descent earlier agains the each of tiny training samples.

By using mini-batch method, we can separate the entire data set like this,
$$X = [x^{(1)}, .. , x^{(1000)} | x^{(1001)}, .. , x^{(2000)}|....] = [x^{\{1\}},x^{\{2\}},.. ]$$
, where each mini-batch has 1000 samples and we have 5000 mini-batches in total.


## 1.1 Mini-batch gradient descent
---
- a mini-batch 't' : $$x^{\{t\}}, y^{\{t\}}$$, where x is training input and y is corresponding output label
- Algorithm
	Let's assumes that we have five-thousands mini-batches.
```
for t = 1..5000
	forward propagation on x^{t}
		Z^[1] = W[1]*x^{t} + b^[1]
		A^[1] = g^[1](Z^[1])
		...
		A^[L] = g^[L](Z^[L]) 
	compute cost 
		J^{t} = 1/1000 * sum(L(y_hat, y)) + regularization_term
	Backprop to compute gradient cost J^{t}
	Update cost
		W^[L] = W^[L] - alpha*dW^[L]
		b^[L] = b^[L] - alpha*db^[L]
```

Instead of one single pass of gradient descents, now we have separate 5000 gradient descents with mini-batche method.


## 1.2 understanding mini-batch gradient descent
---

- mini-batch version is not monotonically decreasing because we train different batches on every iteration.
![](../../../../images/Pasted%20image%2020240131095828.png)

- choosing mini-batch size
	- when the size is equal to m (extremely large)
		$$(X^{\{1\}}, Y^{\{1\}}) = (X, Y)$$
	- when the size is equal to 1 (extremely small, called as stochastic gradient descent)
		-> every examples is its own mini-batch
		$$(X^{\{1\}}, Y^{\{1\}}) = (x^{(1)}, y^{(1)})...$$
Following the above extreme cases, we can see how they actually works in this image.
- blue one - a batch gradient descent
- purple one - stochastic gradient descent
	- this will live around the minimum, but won't converge to global minimum.
![](../../../../images/Pasted%20image%2020240131100508.png)

A good pratice to choose mini-batch size is to select a size in-between of stochastic and batch gradient descent (not too large , not too small.)
 -> this leads to the fastest learning performance.

Few tips to choose a mini-batch size,
- If we have small training set -> just use batch gradient descent (m <= 2000)
- typical mini-batch size follows power of 2 (64, 128, 256, 512,... )
- Make some mini-batches fit in CPU / GPU device memory.



# 2. Exponentially weighted averages
---

Example ) temparature in London
- x-axis : days in a year
- y-axis : corresponding temparature to each days
![](../../../../images/Pasted%20image%2020240131101535.png)

If we compute local average, we can set these formula
$$v_t = \beta*v_{t-1} + (1-\beta)*\theta_{t}$$, where $v_t$ is approximately averaged over $\frac{1}{1-\beta}$ days temparature.
When the $\beta$ is equal to 0.9,  we can get a moving average, which is plotted on red line.
![](../../../../images/Pasted%20image%2020240131101935.png)

The more $\beta$ becomes smaller, the more the data plot becomes noisy. (It's because the influence of current temparature is reduced when we increase the $\beta$)


## Understanding exponentially weighted averages
---


Assuming that we have a recurrence equation : $$v_t = \beta v_{t-1} + (1-\beta)\theta_t$$
We can expand this equation until we encounter the base case. 
ex) $\beta$ = 0.9
- $v_{100} = 0.9v_{99} + 0.1\theta_{100}$
- $v_{99} = 0.9v_{98} + 0.1\theta_{99}$
Then,
$$v_{100} = 0.9v_{99} + 0.1\theta_{100} =0.9(0.9v_{98} + 0.1\theta_{99})  + 0.1\theta_{100} = (0.9)^2v_{98} + (0.9)(0.1)\theta_{99} + 0.1\theta_{100}$$
and so on.

This is why it is called exponentially weighted averages because as you can see we multiply weights whenever expand the recurrence equation.

Therefore, When we set beta close to zero, it means that we give high weight to current temparature. On the other hand, if we set beta close to one, we want to get more averaged value.
![](../../../../images/Pasted%20image%2020240201093756.png)



## Bias correction
![](../../../../images/Pasted%20image%2020240201094410.png)
When we set $\beta$ as 0.98 , we expect that we are going to get green curve.
But what we actually get is purple curve because of bias.

Because we set base case($v_0$) to 0 initially, we have bad estimates at the beginning of days.
A new equation comes up to solve this problem in initial phases.
$$\frac{v_t}{1-\beta_t}$$
- When 't' is small, we remove bias by multiplying a number bigger than 1
- When 't' is large, the $\beta_t$ converges to zero and we can ignore the bias correction gradually because denominator will become 1


# 3. Gradient descent with momentum

Compute an exponentially weighted average of our gradients and then use that gradient to update our weights.

In standard gradient descent method, we might get this movement on the contour by updating parameters.
![](../../../../images/Pasted%20image%2020240201095517.png)
In vertical axis, we want our learning to be slower because we don't want to overshoot according to axis.
But in horizontal axis, we want the learning to be faster because we want to approach global minimum faster.

The momentum algorithm follows :
- on interation t
	- compute $dw, db$ on current mini-batch
	- $v_{dw} = \beta v_{dw}+(1-\beta)dw$
	- $v_{db} = \beta v_{db}+(1-\beta)db$
	- Then, To update parameters, we use $v_{dw}, v_{db}$ , not $dw, db$
	- $W = W - \alpha v_{dw}$. $b = b - \alpha v_{db}$

How we can obtain the result that we want to get by using this momentum algorithm?
- For the vertical axis, we are averaging previous gradient descent. So we can get a slower learning about it gradually
- For the horizontal axis, every gradient descents is going to same direction and it means that the weighted averages is still big. -> faster learning on the axis.


# 4. RMSprop

The algorithm is called 'Root Mean Squared prop'.
Let's take a same example from momentum algorithm explanation
![](../../../../images/Pasted%20image%2020240201101347.png)
, wheere vertical axis is corresponded to w and horizontal axis for b.

The RMSprop follows :
- on interation t :
	- compute dw, db on current mini-batch
	- $S_{dw} = \beta S_{dw} + (1-\beta)dw^2$, where square of derivatives is element-wise operation.
	- $S_{db} = \beta S_{db} + (1-\beta)db^2$
	- $W := W - \alpha\frac{dw}{\sqrt{S_{dw}}}$
	- $b := b - \alpha\frac{db}{\sqrt{S_{db}}}$

To get desirable results on the parameters, we need smaller 'dw'(leads to faster leanring on horizontal axis) and larger 'db' based on the update equations .

If we take one slope from updating steps, we can get a slope like this
![](../../../../images/Pasted%20image%2020240201101922.png)
As you can see, The delta of vertical axis is quite bigger than the amount of horizontal axis.
In our example, $db^2$ will be relatively large and it makes $S_{db}$ larger either, whereas $dw^2$ will be smaller and $S_{dw}$ is smaller.

So we update a parameter 'b' with 'db' divided by larger denominator and this leads to slower learning on the vertical axis.
 
From RMS prop, we can improve learning paths like green-colored path.
![](../../../../images/Pasted%20image%2020240201102222.png)


# 5. Adam optimization algorithm
Adaptive momentum estimation algorithm put 'momentum' and 'RMSprop' in together.

Algorithm :
- initialzie $v_{dw}, S_{dw}, v_{db}, S_{db}$ as 0
- on iteration t :
	- compute $dw, db$ using current mini-batch
	- update variables using hyper parameters '$\beta_1$ and $\beta_2$'
		- $v_{dw} = \beta_1v_{dw} + (1-\beta_1)dw$ , $v_{db} = \beta_1v_{db} + (1-\beta_1)db$ (momentum update)
		- $S_{dw} = \beta_2S_{dw} + (1-\beta_2)dw^2$ , $S_{db} = \beta_2S_{db} + (1-\beta_2)db^2$ (RMSprop update)
	- Bias correction 
		- $$v_{dw}^{corrected}=\frac{v_{dw}}{1-\beta_1^t}, v_{db}^{corrected}=\frac{v_{db}}{1-\beta_1^t}$$
		- $$S_{dw}^{corrected}=\frac{S_{dw}}{1-\beta_2^t}, S_{db}^{corrected}=\frac{S_{db}}{1-\beta_2^t}$$
	- Update parameters
		- $$W := W - \alpha \frac{v_{dw}^{corrected}}{\sqrt{S_{dw}^{corrected}}+\epsilon}$$- $$b := b - \alpha \frac{v_{db}^{corrected}}{\sqrt{S_{db}^{corrected}}+\epsilon}$$

In Adam optimization algorithm, we have bunch of hyper parameters
- $\alpha$ : needs to be tuned
- $\beta_1$ : 0.9 is default choice
- $\beta_2$ : 0.999 is default choice
- $\epsilon$ : $10^{-8}$ is default

# 6. Learning rate decay

Motivation for learning rate decay implementation

- blue colored path : fixed value learning rate
	The path will live relatively bigger region around global minimum.
- green colored path : learning rate decay version
	This path will live tighter region around the minimum
![](../../../../images/Pasted%20image%2020240202093350.png)

The idea is take bigger steps at the initial phases of the learning, then gradually slows down the learning rate while the learning is on processing.

How to implement?
- 1 epoch = 1 pass through the data
	- ex) If we have 5000 thousands mini-batch with 1000 samples for each batch, 1-epoch means we pass through the whole mini-batches one time.
- learning rate decay formula :

$$\alpha=\frac{1}{1+(decayRate)\times(epochNumber)}\alpha_0$$
- other formulas
	- exponential decay
	- constant over square root of number of epoch as multiplier
	- consant over square root of number of mini-batch as multiplier
	- discrete staircase 
or 
- manual decay - observe the learning of model and manually modify learning rate.

# 7. The problem of local optima

For example, 
- vertical axis : cost function
- two horizontal axes : w1 , w2 parameters
Traditionally zero value of gradient descent is considered as local optima or global optima in the dimension.
![](../../../../images/Pasted%20image%2020240202094507.png)

But it actually turns out the most point of zero gradients are not local optima, but saddle points like below. 
The more the dimension of cost function becomes high, the more the chance of saddle points occurence becomes high.
![](../../../../images/Pasted%20image%2020240202095717.png)

So local optima problem in low-dimension is not actual problem that we should solve.
The problem is  'plateaous'.
Plateau is a region where the derivative is close to zero for a long time.

Because the gradients already quite small in plateau region, it will take too long to get off the plateau.
![](../../../../images/Pasted%20image%2020240202100312.png)
Luckily, optimization algorihtms like Adam can improve learning speed in this case.

