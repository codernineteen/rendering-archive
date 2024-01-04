 
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
	- number of training set / test set : $$m_{training} , m_{test}$$
	- put all training examples into compact notation with a matrix 'X' :  $$X = \begin{bmatrix} 
	  \vdots & \vdots & ... & \vdots\\ 
	  x^{(1)} & x^{(2)} & ... & x^{(m)}\\ 
	  \vdots & \vdots & ... & \vdots\\ 
	  \end{bmatrix}$$ , where width is number of training examples and height is dimension of each of those.
	 To train neural network easily, it is also a good way to stack y-label in a column major.
	 $$Y = \begin{bmatrix} 
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
$$J(W, b) = \frac{1}{m}\sum_{i=1}^{m}L(\hat{y^{(i)}, y^{(i)}}) = -\frac{1}{m}\sum_{i=1}^{m}(y^{(i)}\log\hat{y^{(i)}} + (1-y^{(i)})\log(1-\hat{y^{(i)}}))$$
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
$$W := W-\alpha\frac{\partial{J(W, b)}}{\partial{W}}$$
$$b := b-\alpha\frac{\partial{J(W, b)}}{\partial{b}}$$

# Derivatives
a thing to remember here :
 "Derivatives is just a slop of function."


# Computation Graph
![[Pasted image 20240102191554.png]]


# Computing Derivatives
![[Pasted image 20240104075518.png]]

# Derive gradient descent for logistic regression
using computation graph

The below image shows how we get a loss function for logistic regression by using computation graph.
![[Pasted image 20240104081427.png]]

First, we do derivates of Loss function against a.
The "da" variable denotes that final ouput about intermediate quantities. 
![[Pasted image 20240104081635.png]] Utilizing derivative formulas, we can derive the $\frac{dL(a,y)}{da}$
![[Pasted image 20240104082201.png]]

By repeating above back propagation process, finally we can get $dW_1$ ,$dW_2$ and $db$. Then we can use it to update each of values to optimize them.

Let's expand this one training example gradient descent to m-training examples case.
- recap 
$$J(W, b) = \frac{1}{m}\sum_{i=1}^{m}L(\hat{y^{(i)}, y^{(i)}}) = -\frac{1}{m}\sum_{i=1}^{m}(y^{(i)}\log\hat{y^{(i)}} + (1-y^{(i)})\log(1-\hat{y^{(i)}}))$$

If we want to take partial derivatives about J(w, b) in respect to w1, it will be :
$$\frac{\partial{J(W, b)}}{\partial{W1}} = \frac{1}{m}\sum_{i=1}^{m}\frac{\partial{L(\hat{y^{(i)}, y^{(i)}})}}{\partial{W_1^{(i)}}}$$
Because we already solved how the derivatives related to W1 is calculated in one training example, we just need to repeat the same process about m-training examples.

- Professor's derivation
![[Pasted image 20240104084254.png]]
One problem is here. That is it requires two for-loops in the calculation process.
The one is for training examples and the other is for features of each training example.
In deep learning, we handle a huge amount of data and O(n^2) time complexity is inefficient in regard to this amount of data.
-> **vectorization** technique comes in handy

# Vectorization

For example,
There is an equation $$z=w^Tx + b$$ When we calculate it non-vectorized form, it may follow this algorithm using loop.
```plain
z = 0
for i in range(n,x) :
	z += w[i] * x[i]
z += b
```


In python, we can vectorize this equation using numpy library.
```python
z = np.dot(w, x) + b
```

Now we are curious how it can be faster than iterative approach. Does it really avoid looping in calculation?

Both of CPU and GPU can use SIMD(single instruction multiple data). When we use numpy library in python, we can utilize this kind of instruction because we can handle each of elements in vector independently.
Therefore, it is inevitable to use iteration in arithematic operation on computer, but if we use SIMD approach , we can run it much faster through parallelism of processors.


- more examples of vectorization
1. matrix multiplication
 - without vectorization
```
	     u = Av # A is a matrix and v is a vector
	     ->
	     for i..
		     for j..
			     u[i] += A[i][j] * v[j]
```
- with vectorization
```python
u = np.dot(A, v)
```

2. apply exponential operation on every elements in a vector or matrix
- without
```python
for i in range(n):
	u[i] = math.exp(v[i])
```
- with
```python
u = np.exp(v)
```

With vectorization, not only we can compute operations faster than for-loop, but also can represent statements in a semantic way.

# Optimize Logistic regression

- initial algorithm
```
J = 0, dw1 = 0, dw2 = 0, db = 0
for i = 1 to m:
  z^(i) = W^Tx^(i)+b
  a^(i) = sigmoid(z^(i))
  J += -[y^(i)loga^(i)+(1-y^(i))log(1-a^(i))]
  dz^(i) = a^(i) - y^(i) # dz/da
  dw1 += x1^(i)dz^(i)
  dw2 += x2^(i)dz^(i)
  db += dz^(i)
J = J/m, dw1 = dw1/m, dw2 = dw2/m, db = db/m 
```

- vectorize common operations
```
J = 0, dw = np.zeros(dim(x), 1), db = 0 # vectorize two W parameters to one vector
for i = 1 to m:
  z^(i) = W^Tx^(i)+b
  a^(i) = sigmoid(z^(i))
  J += -[y^(i)loga^(i)+(1-y^(i))log(1-a^(i))]
  dz^(i) = a^(i) - y^(i) # dz/da
  dw += x^(i)dz^(i)
  db += dz^(i)
J = J/m, dw = dw/m, db = db/m 
```


We can further optimize this process.

1. Vectorize forward propagation
To get a predictoin value, we need to compute each of it related to each of samples.
$z^{(1)}=W^Tx^{(1)}+b$, $z^{(2)}=W^Tx^{(2)}+b$, and so on.

How can we compute all Zs and predictions at one step? the answer is trivially vectorize those.
- Z =  $[z^{(1)}, z^{(2)}... ]$
- X =  $[x^{(1)}, x^{(2)}... ]$
- b = $[b,b,...]$
Therefore all the equation can be represented as one equation.
$$Z=w^TX+B$$, where Z and X m-dimensional row vector.

In python statements,
```python
Z = np.dot(w.T, X) + b # w.T means transpose of w 
```
Although variable 'b' is a real number, python has a functionality to broadcast the value to target matrix or vector by expanding the real number automatically.

2. Vectorize backward propagation