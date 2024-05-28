

# Hyperparameter tuning
## Tuning process

Hyperparameters we have :
- learning rate (the most important)
- $\beta$ (2)
- $\beta_1, \beta_2, \epsilon$
- number of layers (2)
- number of hidden units
- learning rate decay
- mini-batch size (2)

When we try hyper parameter tuning, we don't use a grid cell.
Instead, we try that on the random points like right-side image.
![](../../../../images/Pasted%20image%2020240205095915.png)

For example, if we have epsilon and learning rate as hyper parameters, we only have five different learning rate values that we can try.
But even though we change the value of epsilon, it will not impact on the learning process much.
However, if we choose random point, we may have 25 different learning rate which is more various than before.


- coarse to fine
Let's assume that we found the blue circled points work well in our model.
Then, we can zoom in the area of samples and sample randomly again in that area(coarse).
![](../../../../images/Pasted%20image%2020240205100418.png)
## Using an appropriate scale to pick hyperparameters

In related to number of layers and hidden units, uniform random in certain range could be okay.
But it wouldn't work in tuning learning rate.
If we have a range between 0.0001 and 1 and use uniform random method to pick a learning rate in that range, we only have 5% chance to pick a value between 0.0001 and 0.1.

Instead of uniform, we can use log scale in this case.
Below range shows log scale.
![](../../../../images/Pasted%20image%2020240205101220.png)

$$a =\log_{10}{0.001}=-4, b =log_{10}{1}=0$$

If we write this down in python code,
```python
r = -4 * np.random.rand() # [-4, 0]
alpha = np.power(10, r) # [10^-4, 10^0]
```

- hyperparameters for exponentially weighted averages
$\beta$ is usually ranged between 0.9 ~ 0.999 and then we can get the range of (1-$\beta$), which is 0.1~0.001.
After we get a range of (1-$\beta$), we can also apply the log scale here.

Why linear scale is bad?
ex) 
- $\beta$ : 0.900 -> 0.905 is no big deal
- $\beta$ : 0.999 -> 0.9995 has huge impact. 
	The beta implies $\frac{1}{1-\beta}$ days
	0.999 is corresponded to 1000 days, but 0.9995 is corresponded to 5000days.

## Hyperparameter tuning in practice

- re-test hyperparaemters occasionally
- babysitting one model (large data sets, but not a lot of computational resources)
	Watching performance and tuning hyperparameters day by day
- Training many models in parallel
	Try different hyperparameters on models in a parallel way.
![](../../../../images/Pasted%20image%2020240205103737.png)







# Batch Normalization

 - This technique makes hyper parameter search more easier
- Also Enable us to much more easily train very deep networks.

We saw an input normalization in the first week by dividing up inputs with its standard deviation.
But in terms of the deep network, can we still apply this concept?
For example, when we have a deep network like below,
![](../../../../images/Pasted%20image%2020240207114741.png)

**Can we normalize $a^{[2]}$ to train $W^{[3]}, b^{[3]}$ faster?**
-> This is exactly what batch normalization does.(technically normalize 'z', not 'a')

## Implement batch norm

Algorithm :
Given some intermediate values in NN (... , $z^{(i)}, ...$), where $1 \le i \le m$
- Normalize $z^{(i)}$  
	- compute mean
	$$\mu=\frac{1}{m}\sum_{i}z^{(i)}$$
	- Compute variance
	$$\sigma^2=\frac{1}{m}\sum_{i}(z^{(i)}- \mu)^2$$
	- Get a normalized 'z' (we added epsilon for numerical stability because value inside of sqrt can be zero)
$$z^{(i)}_{norm} = \frac{z^{(i)}-\mu}{\sqrt{\sigma^2+\epsilon}}$$

But We don't want the hidden units to always have mean 0 and variance 1.
For different distributions, we modify the equation a little bit
$$\tilde{z^{(i)}}=\gamma*z^{(i)}_{norm}+\beta$$, where $\gamma, \beta$ are learnable parameters .

Previously, we used a set of 'z' to activate neurons.
However now we use $\tilde{z}$ for the activation process.

## Fitting batch norm into a Neural network

Example network)
![](../../../../images/Pasted%20image%2020240207120540.png)

If we apply batch norm,
X (with w1, b1) -> z1 -> batch normalize(with $\beta1$, $\gamma1$) -> $\tilde{z^{[1]}}$ -> activation function -> $\alpha^{[1]}$ -> .... so on.

Now we get additional parameters for every step, 
Parameters : $$W^{[1]}, b^{[1]}, \gamma^{[1]}, \beta^{[1]}, ...$$
Note that the beta is totally different thing that we used in Momentum or Adam algorithm as hyperparmeter.


## Working with mini-batches
In this case , we don't use the whole $X$, but mini-batch $X^{i}$ whose size is smaller than a batch $X$.


A important feature of batch norm is that we can ignore the bias term because it always has no impact on the z by being subtracted with mean of the 'z'.
The equation will be :
$$z^{[l]} = W^{[l]}a^{[l-1]}$$ without bias anymore.


## Implement gradient descent 
```
for t = 1 ... # of mini batches:
	1. compute forward prop on X^{t}
		for each hidden layer , use Batch Norm to replace Z^[l] with z_tilde^[l]
	2. Use back prop to compute dw^[l], dgamma^[l], dbeta^[l]
	3. update parameters with usual formulas
```

By the way, we can still use Momentum, RMSprop or Adam for updating parameters regardless of batch norm.



## Why batch norm works?

With the black box of left-side network, the role of third layer is to take previous activations and find a way to map them to 'y hat(prediction)'.

![](../../../../images/Pasted%20image%2020240207123524.png)

But if we remove the black box, we can know that there are also parameters that affect to activations on second layer.
![](../../../../images/Pasted%20image%2020240207123657.png)

Hence, whenever the paramaters located before second layer changes , the activations on second layer also changes.
This causes a problem called 'covariance shift'.

**Covariate shift**
To understand what the convariate shift, let's take an example.
Assuming that our model learned with the real images of animals to classify whether an given image is a cat or not.
![](../../../../images/Pasted%20image%2020240207123941.png)
In given test set, our classifer is required to classify below images
![](../../../../images/Pasted%20image%2020240207124104.png)

But trivially this doesn't work well because our model learned real image, but test set data is cartoon images.
This difference in distributions between training and testing is called 'covariate shift'.

In our network example, while we make our model learn further, there are possibillities to change values of z.
But our batch norm technique ensure that at least we can keep the 0-mean and 1-variance even though the values changes by forcing the parameter gamma and beta to keep the mean and variance in constant.
![](../../../../images/Pasted%20image%2020240207124529.png)

## Batch Norm as regularization (unintended side-effect)
- Each mini-batch is scaled by the mean/variance computed on just taht mini-batch
- This adds some noise to the values $z^{[l]}$ within that minibatch. So simlar to dropout , it adds some noise to each hidden layer's activations.
- This has a slight regularization effect.


## Batch Norm at test time
- mini-batch normalization recap.
![](../../../../images/Pasted%20image%2020240207125413.png)

At test time , we need separate mean and variance which is exponentially weighted average. (across mini-batches)

- mini-batches : $X^{\{1\}}, X^{\{2\}}, ...$ 
- means for each mini batch at a layer 'l' : $\mu^{\{1\}[l]} = \theta_1, \mu^{\{2\}[2]}=\theta_2,...$
- At test time, for a test example, apply estimated mean and variance by exponentially weighted average.


# Multi-class classification

So far, we've handled only binary classification problem.
But what if we have to classify a thing among multiple classes?

For example,
- baby chick - 3
- dog - 2 
- cat - 1
- none of them - 0
![](../../../../images/Pasted%20image%2020240208094601.png)
We have 4-possible class notated as 'C' and we use softmax layer to classify these multi-class.

- softmax layer
![](../../../../images/Pasted%20image%2020240208094921.png)
Following the given animal iamge, now we have 4-units on output layer 'L'.
So the dimension of activation parameter '$Z^{[L]}$' becomes (4, 1).
Also we use different activation function : 

1. element-wise exponentiation.
$$t=e^{(z[L])}$$
2. normalization 
$$a^{[L]}=\frac{e^{Z{[L]}}}{\sum_{i=1}^{4}t_i}$$

Previously, our sigmoid activation function made an output which is just a number.
But in the softmax activation function, the dimension of both output and input is equal to (n, 1) where n is the number of units on the ouput layer.

- Examples of softmax (regression approach)
We can think of the softmax classifier as generalized version of logistic regression.
![](../../../../images/Pasted%20image%2020240208100107.png)

## Training a softmax classifier
The name of 'softmax' is orignated as a contrast of 'hard max'.
In the previous section, we saw a 4-classes classfier and we exponentiated the values of 'z' to make a temporary vector 't'.
![](../../../../images/Pasted%20image%2020240208100415.png)
Then we activated them by normalizing the Z vector.
![](../../../../images/Pasted%20image%2020240208100532.png)

You can see that the higest value in Z still has the highest probability in activated vector.
If we used 'hard max', it would be like this.
$$\begin{bmatrix} 1\\0\\0\\0\end{bmatrix}$$

The other surprising fact of softmax regression is that it is just logistic regression when the C is equal to 2 (binary classes).

## Loss function of softmax

Ex)
- ground true label and prediction.
$$y=\begin{bmatrix}0\\1\\0\\0\end{bmatrix}, a^{[l]}=\hat{y}=\begin{bmatrix}0.3\\0.2\\0.1\\0.4\end{bmatrix}$$
- loss function
$$L(\hat{y}, y)=-\sum_{i=1}^{4}y_j\log{\hat{y}_j}$$
To understand this loss function intuitevly, let's replace certain numbers with the variables in the funciton.
Based on the ground true labels and the form of loss function, all of the label is equal to zero except for $y_2$.
So we can say that $L(\hat{y}, y)=-\sum_{i=1}^{4}y_j\log{\hat{y}_j} = -y_2\log{\hat{y}_2} = -log{\hat{y}_2}$ .
We want to make this loss small and the only way to do that is to make $\hat{y}_2$ big.

How about the entire cost function ?
$$J(w^{[1]}, b^{[1]},...)=\frac{1}{m}\sum_{i=1}^{m}L(\hat{y^{(i)}},y^{(i)})$$

Finally, the key difference in gradient descent is also loss layer. (skipped derivation)
$$dz^{[l]}=\hat{y}-y$$



# Deep learning frameworks

## A tips for choosing deep learning frameworks
- ease of programming (development, deployment)
- running speed
- truly open - open source with good governance

## Tensorflow

### 1. motivation for tensorflow
Assume that we have a cost function :
$$J = w^2 - 10w +25$$
In our manual implementation for NN, we need to take derivative of it ourselves.

But Let's see how we can simplify this in tensorflow library.
In tensorflow, we only need to care about the steps of forward propagation because the library automatically find a way to propagate backward with given cost function.

```python
import numpy as np
import tensorflow as tf

w = tf.Variable(0, dtype=tf.float32)
optimizer = tf.keras.optimizers.Adam(0.1) # adam optimization with 0.1 learning rate

def train_step() :
	with tf.GradientTape() as tape: # gradient tape records a sequence of operation and play the tape backward by visiting the order of operations in a reverse direction
		cost = w ** 2 - 10 * w + 25
	trainable_variables = [w]
	grads = tape.gradient(cost, trainable_variables) # replay tape
	optimizer.apply_gradients(zip.(grads, trainable_variables)) # the standard zip function enclose two lists into a list whose element is a pair of elements in given two lists.

```


### 2. tensorflow with training data
```python
w = tf.Variable(0, dtype=tf.float32)
x = np.array([1.0, -10.0, 25.0], dtype=np.float32) # x controls coefficients of cost function
optimizer = tf.keras.optimizers.Adam(0.1) # adam optimization with 0.1 learning rate

def training(x, w, optimizer) :
	def cost_fn():
		cost = x[0]*w ** 2 + x[1] * w + x[2]
	for i in range(1000) : # num iterations
		optimizer.minimize(cost_fn, [w])
	return w

w = training(x,w,optimizer)
```


# Reference
- course material
https://www.coursera.org/learn/deep-neural-network/lecture

- covariate shift image
https://ko.d2l.ai/chapter_deep-learning-basics/environment.html