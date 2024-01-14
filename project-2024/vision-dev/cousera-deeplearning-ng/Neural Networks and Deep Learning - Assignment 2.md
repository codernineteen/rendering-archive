
# Goal
To build a model to fit data which have red(y=0) and blue(y=1) labels.
-> Classifier to define regions as either red or blue.
![](../../../images/Pasted%20image%2020240114162007.png)


- features(matrix) : X1, X2
- label(vector) : red, blue

# 1. shape of X and Y
- X : (2, 400)
- Y : (1, 400)
- num of training examples (m) : 400

# 2. Neural network model

General methodology to build a neural network :
- Define the neural network structure

![](../../../images/Pasted%20image%2020240114162713.png)

- Initialize the model's parameters
	- np.random.randn(a,b) * 0.01 for weight matrix
	- np.zeros((a,b)) for bias.
- Loop
	- forward propagation
		- tanh function is given as 'np.tanh'
$$Z^{[1]} =  W^{[1]} X + b^{[1]}\tag{1}$$
$$A^{[1]} = \tanh(Z^{[1]})\tag{2}$$
$$Z^{[2]} = W^{[2]} A^{[1]} + b^{[2]}\tag{3}$$
$$\hat{Y} = A^{[2]} = \sigma(Z^{[2]})\tag{4}$$
	- compute loss
		$$J = - \frac{1}{m} \sum\limits_{i = 1}^{m} \large{(} \small y^{(i)}\log\left(a^{[2] (i)}\right) + (1-y^{(i)})\log\left(1- a^{[2] (i)}\right) \large{)} \small\tag{13}$$
	- backward propagation to get the gradients
	![](../../../images/Pasted%20image%2020240114164925.png)
	- update parameters (gradient descent)
	General gradient descent rule : $\theta = \theta - \alpha \frac{\partial J }{ \partial \theta }$ where $\alpha$ is the learning rate and $\theta$ represents a parameter.

What we did here :
- Built a complete 2-class classification neural network with a hidden layer
- Made good use of a non-linear unit
- Computed the cross entropy loss
- Implemented forward and backward propagation
- Seen the impact of varying the hidden layer size, including overfitting.