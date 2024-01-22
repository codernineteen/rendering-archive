
Quick recap :
- logistic regression
![](../../../../images/Pasted%20image%2020240115124929%201.png)

- 1 hidden layer neural network
![](../../../../images/Pasted%20image%2020240115125004%201.png)

Then, what is deep neural network?
We can say that deep neural network is a network having many hidden layers.
Technically a shallow network like logistics regression is also neural network, but researchers found out that there are features that only deep neural network can obtain.
![](../../../../images/Pasted%20image%2020240115125135%201.png)

# Notation in deep neural network
Ex) 4 layer NN
- L : number of layers
- $n^{[l]}$ : number of units in the specific layer
	- ex) $n{[2]}=5$,  $n{[0]}=n_x=3$,
- $a^{[l]}$ : activation in layer l
	- $a^{[l]}=g^{[l]}(z^{[l]})$
- input X = $a^{[0]}$
- $W^{[l]}$ : weighs for $z^{[l]}$
- $a^{[L]}=\hat{y}$ where, L is upper case.
	
![](../../../../images/Pasted%20image%2020240115125357%201.png)


# Forward propagation in deep NN

General forward propagation follows for each training examples:
$$z^{[l]}=W^{[l]}a^{[l-1]}+b^{[l]}$$
$$a^{[l]}=g^{[l]}(z^{[l]})$$
If we vectorize this for each training examples,
$$Z^{[l]}=W^{[l]}A^{[l-1]}+b^{[l]}$$
$$A^{[l]}=g^{[l]}(Z^{[l]})$$
Although we have explicit for-loop against number of layers, it is totally okay to have it in this case.


# Getting matrix dimensions right (very important)
For example, when we have 5 layer NN like below,
![](../../../../images/Pasted%20image%2020240115132403%201.png)

The dimensions of each parameters and variables follow :
![](../../../../images/Pasted%20image%2020240115132446%201.png)


# Why deep representations work well

For the case of face detection, The input will be an image about a human face.
![](../../../../images/Pasted%20image%2020240115132715%201.png)

Then, we can think of the first hidden layer as a feature detector or an 'edge' detector.
![](../../../../images/Pasted%20image%2020240115132812%201.png)

Now the second layers grouping the edges to form part of faces.
![](../../../../images/Pasted%20image%2020240115132959%201.png)

Finally last hidden layer creates a various of faces by using the parts of the face.
![](../../../../images/Pasted%20image%2020240115133105%201.png)

The whole process will be
![](../../../../images/Pasted%20image%2020240115133121%201.png)


The intuition for this example is to find simpler things first and compose it together to form more complex one.

Another statements about why deep NN works well :
"There are functions you can compute with a small L-layer deep NN that shallower networks require exponentionally more hidden units to compute"

For example, we want to compute XOR operations for each of features from x1 to xn
$$Y = x_1 \mathbin{\oplus} x_2 ...\mathbin{\oplus} x_n$$
If we solve this using tree structure allowing multiple hidden layers, the depth of tree will be O(log(n)) and this is not that large.
On the other hand, If we are not allowed to use multiple hidden layers,  the complexity goes up to O(2^n) when we have 1 hidden layer.


# Building blocks of Deep neural networks

When we have such a neural network, let's assume that we're one a layer number 'L'
![](../../../../images/Pasted%20image%2020240117091652.png)
Then for forward prop, the input will be $a^{[l-1]}$ and output will be $a^{[l]}$ and $cache(z^{[l]})$
At the same time, the input will be $da^{[l]},cache(z^{[l]})$ and output will be $da^{[l-1]},dw^{[l]},db^{[l]}$ for backward prop.
- drawing description
![](../../../../images/Pasted%20image%2020240117092212.png)

If we apply above process for whole network, 
![](../../../../images/Pasted%20image%2020240117092552.png)
 and Along wtih cache z , we'll cache w, b parameters either for implementation purpose.
 

# Forward propagation and Back propagation

- Forward
	$$Z^{[l]}=W^{[l]}A^{[l-1]}+b^{[l]}$$
	$$A^{[l]}=g^{[l]}(Z^{[l]})$$
	,where $a^{[0]}$ is equal to an input feature $x$

- Backward
	$$dz^{[l]}=da^{[l]}*g\prime^{[l-1]}(z^{[l-1]})$$
$$dw^{[l]}=dz^{[l]}a^{[l-1]^T}$$
$$db^{[l]}=dz^{[l]}$$
$$da^{[l-1]}=w^{[l]^T}dz^{[l]}$$
With vectorization,
$$dZ^{[l]}=dA^{[l]}*g\prime^{[l-1]}(Z^{[l-1]})$$
$$dW^{[l]}=\frac{1}{m}dZ^{[l]}A^{[l-1]^T}$$
$$db^{[l]}=\frac{1}{m}np.sum(dZ^{[l]},axis=1,keepdims=True)$$
$$dA^{[l-1]}=W^{[l]^T}dZ^{[l]}$$
In terms of forward propagation recursion, the input is 'x', Then what is the input for back prop?
It will be $da{[l]}$, which is derivatives of $\hat{y}$.
For example, if we use logistc regression,
$$\frac{d\hat{y}}{da}=-\frac{y}{a}+\frac{1-y}{1-a}$$

# Parameters vs Hyper Parameters

We've seen parameters : $w^{[l]}, b^{[l]}$
About hyper parameters : learning rate $\alpha$ , # of iterations, # of hidden layers, # of hidden units, choice of activation functions.
All of parameters that is not directly relate to computation forward/backward prop itself is called 'hyper parameters'.
Because they somewhat control the result of parameters, we need to adjust them appropriately during the empirical process.
![](../../../../images/Pasted%20image%2020240117101020.png)






# Reference
https://www.coursera.org/learn/neural-networks-deep-learning