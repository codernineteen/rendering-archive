

# What is neural network

In logistic regression, we simply stacked input parameters to one next step.

However, in neural network, we have a stack of node for the next step while we have different weights and bias for each nodes. (the stack of nodes is called 'layer').

- logisitc regression
![logistic regression](../../../../images/Pasted%20image%2020240108083407.png)

- simple neural network
Unlike logistic regression, the 'a' value computed by sigmoid(z) is used again to compute another $z^{[2]}$ -> repeating two times of logistic regression
![](../../../../images/Pasted%20image%2020240108083427.png)

# Neural network representation
- leftmost layer : input layer
- intermediat layer : hidden layer
- rightmost layer : output layer
The word 'hidden' means that it is a layer that is unobservable in training set.

- $a^{[0]}=X$ : a stands for 'activation'. this input layer passes on the value x to hidden layer
- $a_{i}^{[1]}$ : first hidden layer 
- $a^{[2]}$ : $\hat{y}$
![](../../../../images/Pasted%20image%2020240108084110.png)

*note* : we don't count input layer in neural networks, so it is "2 layer NN" in above case.

# Computing a neural networks's output

Logistic regression is composed of two steps in the middle node.
- compute z by linear equation
- compute a using sigmoid function
![](../../../../images/Pasted%20image%2020240108085419.png)

In neural networks, we do same computation, but lots of times.
For example, we want to compute logistic regression against node 2 in first hidden layer.
We use related parameters to the second node in first hidden layer.
Regardless of which nodes we are computing, superscript notation is always same about same hidden layer.
![](../../../../images/Pasted%20image%2020240108085533.png)

Therefore, we are going to get four equations by following same steps for other nodes in the hidden layer.
To compute it efficiently without for-loop, we can also vectorize this linear equations.

- $z_1^{[1]}=w_1^{[1]}+b_1^{[1]}$, $a_1{[1]}=\sigma(z_1^{[1]})$
- $z_2^{[1]}=w_2^{[1]}+b_2^{[1]}$, $a_2{[1]}=\sigma(z_2^{[1]})$
- $z_3^{[1]}=w_3^{[1]}+b_3^{[1]}$, $a_3{[1]}=\sigma(z_3^{[1]})$
- $z_4^{[1]}=w_4^{[1]}+b_4^{[1]}$, $a_4{[1]}=\sigma(z_4^{[1]})$
-> vectorize ->

![](../../../../images/Pasted%20image%2020240108090625.png)

we can write it again as a vectorized formation.
- $z^{[1]}=W^{[1]}x+b^{[1]}$, where z, W, b are matrices that hold their components
- $a^{[1]}=\sigma(z^{[1]})$
- $z^{[2]}=W^{[2]}a+b^{[2]}$, where a comes from first hidden layer.

# Vectorizing across multiple examples
In above, we handle just a single training example.
In this section, we are going to vectorize equations against m-training examples.

From m-training examples, we get :
- $X^{(i)}->a^{[2](i)}=\hat{y^{(i)}}$ - 'i' means i-th training example.
And for every i-th training example, we follow same steps that we did in above section.

Instead of using for-loop , we can vectorize it and the algorithm will be converted into right-side form.
![](../../../../images/Pasted%20image%2020240108092047.png)
*Note that it is a forward propagation algorithm.*

Below image represents how 'Z^[1]' matrix looks like
- horizontally, training examples aligned
- vertically, hidden unit aligned corresponding to a certain training example.
![](../../../../images/Pasted%20image%2020240108092406.png)


Because $X^{[1]}$ is composed of a set $a_i^{[0]}$ , we can rewrite the equation to get a z.
$$Z^{[1]}=W^{[1]}X+b^{[1]}=W^{[1]}A^{[0]}+b^{[1]}$$


# Activiation functions
---
![](../../../../images/Pasted%20image%2020240110081543%201.png)

We've used 'sigmoid' function for activation so far, but there can be better approach.

Instead sigmoid function which is linear, we can replace this with non-linear function 'g'
An activation function is always better than the sigmoid function.

ex) hyperbolic tangent function (tanh)
This function is mathematically shiffted version of sigmoid funciton.
This function is better to center a data whose mean of it is close to zero rather than 0.5(sigmoid's)
**This zero mean makes learning of neural network easier for next layer.**

However, in regard to output layer, it would be better to use sigmoid function in binary classification problem because the Y lable is 0 or 1 and the prediction of our network will be ranged from 0 to 1.

A downside of the both functions is that their gradient changes very slow when the 'z' is very big or small.

One of very popular activation function in machine learning is 'RELU' activation funcition.
It has zero value output when z is smaller than zero and 'y=x' form when z is greater or equal than zero.
In practice, neural networks will often learn much faster with these activation function because the gradients has huge difference according to its 'z' value.

**Common advice about which actviation function we should choose :**
"Try them all to validate which of those fit into your problem best"

# Why neural network need a non-linear activation function?

What if we don't have any kind of activation function in our network?
$$a^{[1]} = z^{[1]} = W^{[1]}X + b^{[1]}, a^{[2]} = z^{[2]} = W^{[2]}a^{[1]} + b^{[2]}$$ Then,
$$a^{[2]} = W^{[2]}(W^{[1]}X + b^{[1]}) + b^{[2]}=W^{[2]}W^{[1]}X + (b^{[1]}+b^{[2]})$$
The above equation is what we are going to get if we use 'linear activation' function.
In a huge neural networks, we have a lot of hidden layers.
But because the function is linear, no matter how many hidden layers we have in the network, it is always same with having just a single hidden layer.
-> less useless..


# Implement back propagation

## Derivatives of activation function
---

1. sigmoid activation function
$$g(z)=\frac{1}{1+e^{-z}}, g'(z)=g(z)(1-g(z))$$
For example, 
when z = 10 , g' is about 0
when z = -10,  g' is about 0
when z = 0, g' is about 1/4

2. Tanh function
$$g(z)=tanh(z), g'(z) = 1-(tanh(z))^2$$

3. ReLU and Leaky ReLU
- ReLU
$$g(z)=max(0, z),g'(z)=\begin{cases}&0&\text{if}z<0\\&1&\text{if}z>0\end{cases}$$
Note that the case of 'z=0' is undefined.

- Leaky ReLU
$$g(z)=max(0.01*z, z),g'(z)=\begin{cases}&0&\text{if}z<0.01\\&1&\text{if}z>0\end{cases}$$


## Gradient descent for neural networks
---
Let's overview the process of gradient descent.
- params : $W^{[1]},b^{[1]},W^{[2]},b^{[2]}$
- dimension of each params are $(n^{[1]}, n^{[0]}),(n^{[1]},1),(n^{[2]},n^{[1]}),(n^{[2]},1)$
	It is recommended to initialize these parameters randomly, not zeros.
- Gradient descent follows :
	- Repeat
	```
	Compute predictions(y hat_i)
	dW^[1] = dJ/dw^[1], db^[1] = dJ/db^[1], ...
	W^[1] = 1 - (learning_rate)*dW^[1]
	b^[1] = 1 - (learning_rate)*db^[1]
	// same for W^[2], b^[2]
	```

Then, Forward propagations are :
![](../../../images/Pasted%20image%2020240111083008.png)

After this, we can do back propagation following the computation graph in a reverse way
![](../../../images/Pasted%20image%2020240111083353.png)
Note that computation of dz^1 use element-wise product operation about same dimension of two matrices.

# Random initialization
Intializing W with all zeros can cause a problem.
When we initialize W with zeros, hidden units in hidden layer become completely identical(symmetric).
Both of hidden units start to compute same function, also they have same influences on output unit.
By induction approach, this will be kept on every iterations because the first iteration turned out that it is true.
By the way, it is okay to initialize bias as zeros.

Solution is to use randomization.
ex) `W_1 = np.random.randn((2,2)) * 0.01` -> here we multiplied 0.01 because we want to avoid z become so huge or small when we use sigmoid or tanh function for activation.

# Reference
https://www.coursera.org/learn/neural-networks-deep-learning