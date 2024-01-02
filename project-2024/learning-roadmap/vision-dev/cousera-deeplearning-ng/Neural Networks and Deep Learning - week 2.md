 
In traditional way, we handle m-training set using for loop
In neural network, we are goint to handle this without loop (How?)
Q. What is forward / back propagation and why it needed?
	- Explain using logistics regression

# Binary classification
Logisitc regression is a technique to solve binary classification. (ex. Is this a cat in an image?)
![[Pasted image 20240102123406.png]]

Before we start, let's see basic notations first.
	- training sample : (x, y)
	- feature vector : $$x \in R^{dim_X}$$
	- y : label that has a value zero or one.
	- m - training example : $$(x_1, y_1), (x_2, y_2),...,(x_n, y_n) $$
	- number of training set / test set : $$ m_{training} , m_{test} $$
	- put all training examples into compact notation with a matrix 'X' :  $$ X = \begin{bmatrix} 
	  \vdots & \vdots & ... & \vdots\\ 
	  x^{(1)} & x^{(2)} & ... & x^{(m)}\\ 
	  \vdots & \vdots & ... & \vdots\\ 
	  \end{bmatrix}$$ , where width is number of training examples and height is dimension of each of those.
	 To train neural network easily, it is also a good way to stack y-label in a column major.
	 $$ Y = \begin{bmatrix} 
	  \vdots & \vdots & ... &\vdots\\ 
	  y^{(1)} & y^{(2)} & ... & y^{(m)}\\ 
	  \vdots & \vdots & ... & \vdots\\ 
	  \end{bmatrix}$$


# Logistic regression

Back to the cat image classification problem,  With given X, Assume we want to a predicted output vector $\hat{y} = p(y=1|x)$ .
When there is two parameters $w \in R^{n_x}$ , bias $b \in R$, One possible idea to solve this problem is a 'linear equation' such as $\hat{y}=w^Tx+b$.
But this is wrong because it can be over 1 while all of probabilities is under 1.
So we can transform this linear equation into sigmoid function.
$$\hat{y}=\sigma{(w^Tx+b)}$$A sigmoid function graph looks like below:
![[Pasted image 20240102130533.png]]
, where $\sigma(z)=\frac{1}{1+e^{-z}}$
When z become so large, e^-z converges to zero, so the value of sigmoid function will converges to one.
In opposite case , the function converges to zero.

**The key point of logistic regression is to train two parameters w and b so that y hat can has good estimate to be 1's propbability**

Then, How we can change this 'W' and 'b' parameters? we can do that by defining '**cost function**'


# logistic regression cost function
	  $$\hat{y^{(i)}} = \sigma(W^Tx^{(i)} + b)$$
	  where, $\sigma(z^{(i)})=\frac{1}{1+e^-z}$ and the '(i)' denotes that i-th example of training sets.
  
  - loss function( $L(\hat{y}, y)$ ) :  a function to measure how much y_hat is good when the true label is y
	  Regard to loss function, we want to make an error as small as possible.
	$$L(\hat{y}, y) = -(y\log\hat{y} + (1-y)\log(1-\hat{y}))$$
	Following intuition, when y is equatl to 1, the right term is equal to $-\log\hat{y}$.
	-> then, we want to make this '$-\log\hat{y}$' as small as possible
	== want $\log\hat{y}$ as large as possible == want $\hat{y}$ as big as possible.
	
	On the other hand, when y is qual to zero, we want to make $\log(1-\hat{y})$ as big as possible, hence, we want to make small $\hat{y}$

In conclusion, the cost function will be the average of sum of Loss function result on each elements in training examples.
$$
J(W, b) = \frac{1}{m}\sum_{i=1}^{m}L(\hat{y^{(i)}, y^{(i)}}) = -\frac{1}{m}\sum_{i=1}^{m}(y^{(i)}\log\hat{y^{(i)}} + (1-y^{(i)})\log(1-\hat{y^{(i)}}))
$$
**It turns out that the logistic algorithm can be viewed as a very small neural network.**


# Gradient Descent
In this section, we are going to treat how gradient descent algorithm can be used to train or to learn the parameters W on our training sets.

Based on the previous Loss function and Cost function, Our goal is to find W, b that minimizes the cost function. -> optimization problem -> can use derivative approach.

As we can see below, The cost function is shaped as convex form.
![[Pasted image 20240102151545.png]]
Taking the direction of steepest descent, we hope that we can reach the global optimal after few iterations.![[Pasted image 20240102151749.png]] 
Now let's see how a gradient descent algorithm works in regard to one variable 'W'
![[Pasted image 20240102152951.png]]
- $\alpha$ : learning rate

In logistics regression, we want to optimize two parameters 'W' and 'b'.
So we can take a partial derivate approach here to optimize those.
$$
W := W-\alpha\frac{\partial{J(W, b)}}{\partial{W}}
$$
$$b := b-\alpha\frac{\partial{J(W, b)}}{\partial{b}}$$

# Derivatives