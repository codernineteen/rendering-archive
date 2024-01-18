
# Goal
- be able to use non-linear units like ReLU to improve our model
- Build a deeper neural network
- implement an easy-to-use neural network class


# Helper functions
- initialize parameters for each layers.
	- for n-layer NN, dimension of W is ($n_l$, $n_{l-1}$) and dimenstion of b is ($n_l, 1$) where $l$ is current layer.
- forward propagation module
	- combine Linear equation and Activation into a step
		- implement 'linear forward' to compute a forward equation $Z^{[l]}= W^{[l]}\cdot A^{[l-1]} + b^{[l]}$
		 - implement activation to activate 'z' with ReLU or sigmoid
	- stack the (Linear -> Activation) forward functions L-1 time
	- ![](../../../images/Pasted%20image%2020240118110258.png)
	- Note -  for every forward function, there is a corresponding backward function (need to store some values in cache.)
- compute loss
	- follow cross-entropy loss formula
	$$-\frac{1}{m} \sum\limits_{i = 1}^{m} (y^{(i)}\log\left(a^{[L] (i)}\right) + (1-y^{(i)})\log\left(1- a^{[L](i)}\right)) \tag{7}$$
	
- backward propagation module
	![](../../../images/Pasted%20image%2020240118112030.png)
	Suppose that we already have $dZ$,
	- linear backward
	- linear -> activation backward
	- Here are the formulas we need:
$$ dW^{[l]} = \frac{\partial \mathcal{J} }{\partial W^{[l]}} = \frac{1}{m} dZ^{[l]} A^{[l-1] T} \tag{8}$$
$$ db^{[l]} = \frac{\partial \mathcal{J} }{\partial b^{[l]}} = \frac{1}{m} \sum_{i = 1}^{m} dZ^{[l](i)}\tag{9}$$
$$ dA^{[l-1]} = \frac{\partial \mathcal{L} }{\partial A^{[l-1]}} = W^{[l] T} dZ^{[l]} \tag{10}$$
	After that, we stack the linear_activation_backward l-1 times.
	![](../../../images/Pasted%20image%2020240118113707.png)

![](../../../images/Pasted%20image%2020240118104323.png)


# Reference
https://www.coursera.org/learn/neural-networks-deep-learning