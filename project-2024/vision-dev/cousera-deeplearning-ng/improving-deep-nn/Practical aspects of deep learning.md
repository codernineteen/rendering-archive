
# Setting up ML application

 we need to determine :
 - number of layers
 - number of hidden units
 - learning rates
 - activation functions

## Datasets in ML
- Training set : train a model to predict a label based on train inputs
- Development set : cross validation
- Test set : test actual result by comparing prediction with true label
![](../../../../images/Pasted%20image%2020240122123846%201.png)

Setting up ML application
-> set it up into a train, dev and test sets.

- If we have relatively small datasets, it might be okay to follow tranditional partitioning datasets rule, which is "7:3"  or "6:2:2" in general.
- If we have a big data set, it is okay to use much smaller dev and test sets.

A few advices from professor Ng
- **Make sure that dev and test sets come from same distribution.**
	- this may leads to faster learning algorithm
- It might be okay not to have test sets if we don't need unbiased estimate.


## Bias and Variance
For example, In regard to cat classification problem like below,
![](../../../../images/Pasted%20image%2020240123103022%201.png)
Human has roughly zero percent error about all classification problems.
But about the machine, we can think of a few different cases.

1. Train set error - 1% , Dev set error 11%
	In this case, we classfied pretty well in training set, but dev set shows much higher error.
	So we can say that the machine has high variance(over fitted)
2. Train set error - 15%, Dev set error 16%
	In this case, Both of training set and dev set has high error percentage.
	We can say that the machine underfits data
3. Train set error - 15%, Dev set error 30%
	we can think of the case when train set has high error and the dev set has even worse error.
	In this case, the machine has high variance and bias at the same time.
		![](../../../../images/Pasted%20image%2020240123104142%201.png)
		We can see the case which is corresponded to purple line in above image.
		It underfits data because it separate most of samples linearly, but also overfitted to the two samples in the middle of samples.
1. Train set error 0.5%, Dev set error 1%
	This may be the best case that our user is satisfied with because it has low variance and bias.

Note that the optimal error is called as 'Bayes error' and Bayes error can have high variance and bias either.


## Basic Recipe for Machine Learning

Steps to follow :
1. Does our algorithm have high bias ? (related to training data performance)
	If yes, try 'Bigger network' , 'training longer' or 'different NN architectures'.
2. Then, High variance?
	if yes, 'more data',  'regularization' , 'different NN arch', then go to first step again
3. Done
Note that the things that we can try to improve the performance against each of problems can be different. For example, trying more data to improve bias problem may not be helpful.

Additionally, in the modern era of deep learning, we can improve the one among bias and variance problem without hurting much performance of the other.

---



# Regularization

When we observe high variance problem in our network, the first thing that we can try is 'regularization'.

Ex) Logistic regression regularization
$$J(w,b)=\frac{1}{m}\sum_{i=1}^m L(\hat{y^{(i)}}, y{(i)})+\frac{\lambda}{2m}||w||_2^2$$, where $||w||_2^2=\sum_{j=1}^{n_X}w_j^2=w^Tw$ (L2 regularization) and $\lambda$ is reguralization parameter (which is another hyper parameter.)

Because high variance problem is caused by lots of 'w' parameters in general while b is a single number, we can omit the regularization for bias.

Then, How about neural network?
$$J(w^{[1]},b^{[1]},...,w^{[l]},b^{[l]})=\frac{1}{m}\sum_{i=1}^m L(\hat{y^{(i)}}, y{(i)})+\frac{\lambda}{2m}\sum_{l=1}^{L}||w^{[l]}||^2$$
,where 
$$||w^{[l]}||^2=\sum_{i=1}^{n^{[l]}}\sum_{j=1}^{n^{[l-1]}}(w_{i,j}^{[l]})^2$$ (frobenius norm)

And backpropagation will be changed a bit under this regularization
$$dw^{[l]}=(form-backprop)+\frac{\lambda}{m}w^{[l]}$$
And when we update the parameter 'w'
$$w^{[l]} := w-\alpha dw^{[l]}=w^{[l]}-(\frac{\alpha\lambda}{m})w^{[l]}-\alpha(backprop)$$
Because of regularization, we multiply a weight matrix with a number slightly less than 1

# Why regularization reduces overfitting?

1. When lambda going to be very large
Following below equation to update the parameter $w^{[l]}$,
$$w^{[l]} := w-\alpha dw^{[l]}=w^{[l]}-(\frac{\alpha\lambda}{m})w^{[l]}-\alpha(backprop)$$

We can say that a neural network can be simpler because we can shrink weight matrices.
Then it may leads to high bias form like logistic regression
![](../../../../images/Pasted%20image%2020240124111859.png)

2. example of tanh function
![](../../../../images/Pasted%20image%2020240124112206.png)
When lambda is big, it shrink weight matrices.
When Weight matrices are small, z is also relatively small because z follows this equation :
$$z^{[l]}=W^{[l]}a^{[l-1]}+b^{[l]}$$
Then z is centered around origin on its axis -> tanh function becomes linear as it is drawn in above image.

Here is a small tip :
When we don't have regularization on our cost function, the plot of cost funciton will monotonically decrease.
On the other hand, With regularization, we have second term in cost funciton and it may not decrease monotonically.


# Dropout regularization
In dropout regularization, we set probabilities for each layers and we gradually remove the nodes, outgoing and ingoing edges based on its probability.
This technique will cause a much smaller network while it is training data set and we can intuitively know that it may avoid overfitting because it is trained in smaller network.
![](../../../../images/Pasted%20image%2020240124113004.png)

How to implement?
- Inverted dropout
For example, we have 3-layer network and keep-prob is equal to 0.8.
Then,
`d3 = np.random.rand(a3.shape[0], a3.shape[1]) < keep_prob`
`a3 = np.multiply(a3, d3)`
These statements means that we create a random boolean matrix which has value of one with 0.8 chance and value of zero with 0.2 chance based on the dimension of matrix 'a3'.
Then by multiplying the two matrices in an element-wise way, we can update a3 based on keep-prob.
Then, do
`a3 /= keep_prob`  
It's because we removed 20% of components from matrices and we need to invert it as much as the probability to keep expected value.

But when we test our trained model with inverted dropout technique, we just follow normal activation process without randomly removing nodes.
We want to train our model randomly, but not about the prediction result.

# Understanding dropout 

In addtion to previous intuition, here is another intuition for dropout technique.
![](../../../../images/Pasted%20image%2020240124114941.png)
We can't rely on any one feature, so have to spread out weights. By eliminatiing nodes randomly from previous layer, we can spread out the weights.
Spreading out weights shrink the squared norm of the weights.

Another feature of dropout is that we can vary keep-prob for each layers.
Let's take an examle.
- $w^{[1]}$ : 7 x 3 weight matrix
- $w^{[2]}$ : 7 x 7 weight matrix
- $w^{[3]}$ : 3 x 7 weight matrix
Among the weight matrices, $w^{[2]}$ is the biggest weight matrix.
We can set a lower keep-prob for the second weight matrices to spread out weights more than other layers.
![](../../../../images/Pasted%20image%2020240124115352.png)

One downisde of dropout is that we lose a debugging tool for cost function because it is hard to keep well-defined cost function in every iterations.

# Other regularization method

- Data augmentation
Not enough training samples -> transform the dataset we already have
 ex) flipping images. ![](../../../../images/Pasted%20image%2020240124120227.png)
![](../../../../images/Pasted%20image%2020240124120434.png)


- Early stopping
After plotting each results from training set and devlopment set, Find the best iterations and we can just stop iterating earlier than the number of iterations we set before.
The downside of this method is that there is a tradeoff between optimization on cost function and avoiding overfitting.
If we stop iterations to update parameters early, we are not going to great job to reduce cost. But we may avoid overfitting.


---

# Normalizing inputs

With given these two input features,
We can get an expected value of the samples following $$\mu = \frac{1}{m}\sum_{i=1}^{m}x^{(i)}$$
![](../../../../images/Pasted%20image%2020240125105827%201%201.png)

After then, If we plot the deviation of the samples by subtracting expected value from each values, it will be
![](../../../../images/Pasted%20image%2020240125110157%201%201.png)
It seems that a feature x1 has larger variance than x2.
Then let's get a variance of it.
$$\sigma^2=\frac{1}{m}\sum_{i=1}^{m}x^{(i)2}$$
Note that x^(i) is already deviation and the product is element-wise.
After we update x by dividing it with the square root of variance (which is standard deviation)
we will get:
![](../../../../images/Pasted%20image%2020240125111334%201%201.png)


**Why we want to normalize inputs?**
Keeping remind that cost function we've used before,
- without normalization
![](../../../../images/Pasted%20image%2020240125111726%201%201.png)
- with normalization
![](../../../../images/Pasted%20image%2020240125111733%201%201.png)

Because the shape of cost function without normalization is elongated, it may takes longer to find a global minimum depending on start point.
However, it doesn't matter in normalization case because it is insensitive to starting point by forming symmetric shape.

**In conclusion, Normalization can help our learning algorithm to run faster**.
Especially when one feature has much wider range of values than others, it will be helpful a lot.

---

# Vanishing / Exploding gradients

To understand these problem intuitvely, let's take two cases when the weigh matrices are larger/smaller than identity matrix.

- $w^{[l]} > I$ , where $I$ is identity matrix
	In this case, At the layer index 'l', we will get a prediction corresponded to $\hat{y}=w^{[l]}*...*x$.
	Trivially the activations will be exploded when we have a very deep neural network because all the weight matrices are bigger than identity matrix and they are cumulated exponentially.

- $w^{[l]} < I$ , where $I$ is identity matrix
	This is exactly opposite case to above.

This leads us to train our neural network in difficulty because it will take a long time for gradient descent to learn anything.

We can solve this problem partially by carefully choicing the weight matrices.

# Weight Initialization for Deep networks

Let's start from a single layer network first.

![](../../../../images/Pasted%20image%2020240125113651%201%201.png)

By generalizing an equation for 'z', we can get : $$z = w_1x_1 + w_2x_2 + ... + w_nx_n$$ , ignoring bias for now.
If we want to make z smaller when we have a large n, we need to set w_i smaller.

It turns out that by multiplying *a specific variance* to random weights, we can reduce vanishing or exploding gradient descents problem.
$$var(w:)=1/n$$
, where n is number of units in the layer.
When the acivation function is ReLu, it would be better with
$$var(w:)=2/n$$

So we can set a weight matrix randomly in python by following,
$$W^{[l]}=np.random.randn(shape)*np.sqrt(x/n^{[l-1]})$$
,where x is our choice and n^[l-1] is number of units from previous layer.

For other activations, 
- tanh - xavier initialization
	$$\sqrt{\frac{1}{n^{[l-1]}}}$$

---

# Numerical approximation of Gradients

We have a function that is equalt to $f(\theta)=\theta^3$ .
Instead of taking one-sided triangle around our target, we can two-sided triangle, which contains both sided of episilon around our target.
If we compute a slope using tangent formula, we can get $$\frac{f(\theta+\epsilon)-f(\theta-\epsilon)}{2\epsilon}$$
Assuming the epsilon is equal to 0.01, we will get a value of slope '3.0001'.
If we compare this value with true tangent slope on the target(=3), the approximation error is 0.0001, which is really small.
![](../../../../images/Pasted%20image%2020240125115615%201%201.png)

Using the two-sided difference , we can check whether our implementation of the derivative of a certain function is correct **numerically**.


# Gradient checking (debuggin purpose)
1. Take parameters and reshape into a big vector $\theta$
$$J(w^{[1]}, b^{[1]}, ... w^{[l]}, b^{[l]})=J(\theta)$$
2. Take derivatives of parameters and reshape into a big vector $d\theta$

Then our question 'Is $d\theta$ the gradient of $J(\theta)$?'

To answer this question, our gradient check agorithm follows below steps.
For every i,
	$$d\theta_{approx} = \frac{J(\theta_1,...\theta_i+\epsilon,..) - J(\theta_1,...\theta_i-\epsilon,..)}{2\epsilon}\approx d\theta[i]=dJ/d\theta_i$$
and check whether euclidean distance between two vectors is approximatlely equal to epsilon(10^-7)
$$\frac{||d\theta_{approx} - d\theta||_2}{||d\theta_{approx}||_2+||d\theta||_2}\approx10^{-7}$$


# Practical tips on implementation of gradient check

- Don't use grad check in training
- If there's an failure on grad check, start debugging
- remember regularization (we need to include L2-regularization term into our grad check)
- grad check doesn't work with drop out method.
- Run at random initialization



# Reference
https://www.coursera.org/learn/deep-neural-network/