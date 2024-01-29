Given problem ,
- blue dot : Same team hit the ball
- red dot : opponent hit the ball
Our goal : Use a deep learning model to find the positions on the field where the goalkeeper should kick the ball.
![](../../../../images/Pasted%20image%2020240129140116.png)

# 1. Non-regularized model
- train samples
![](../../../../images/Pasted%20image%2020240129140808.png)

- without regularization -> overfitted classifying
![](../../../../images/Pasted%20image%2020240129140828.png)


# 2. L2-regularization
From
$$J = -\frac{1}{m} \sum\limits_{i = 1}^{m} \large{(}\small  y^{(i)}\log\left(a^{[L](i)}\right) + (1-y^{(i)})\log\left(1- a^{[L](i)}\right) \large{)} \tag{1}$$
To:
$$J_{regularized} = \small \underbrace{-\frac{1}{m} \sum\limits_{i = 1}^{m} \large{(}\small y^{(i)}\log\left(a^{[L](i)}\right) + (1-y^{(i)})\log\left(1- a^{[L](i)}\right) \large{)} }_\text{cross-entropy cost} + \underbrace{\frac{1}{m} \frac{\lambda}{2} \sum\limits_l\sum\limits_k\sum\limits_j W_{k,j}^{[l]2} }_\text{L2 regularization cost} \tag{2}$$


If we have three layers, we can represent this in python
```python
L2_regularization_cost = (np.sum(np.square(W1)) + np.sum(np.square(W2)) + np.sum(np.square(W3)))/m * (lambd/2)
```

After we appropriately updated the way of computing cost funciton, we need to reflect the term into backward propagation steps.($\frac{d}{dW} ( \frac{1}{2}\frac{\lambda}{m}  W^2) = \frac{\lambda}{m} W$).

**Observations**:
- The value of 𝜆 is a hyperparameter that you can tune using a dev set.
- L2 regularization makes your decision boundary smoother. If 𝜆 is too large, it is also possible to "oversmooth", resulting in a model with high bias.

What we need to remember :
- A regularization term is added to the cost 
	-> it generates extra term in back propagation
	-> Wegihts end up smaller (weight decay)


- Result
- ![](../../../../images/Pasted%20image%2020240129142413.png)

# Dropout implementation

- A part of dropout impl
	- D1 : a mask matrix to shut down some units in a hidden layer randomly.
	- update A1 : update A1 multiplying by D1 (If a component has value 0, the corresponded location will be set as 0 either)
	- divide A1 by keep_prob(inverted dropout) - keep expectation
```python
D1 = np.random.rand(A1.shape[0], A1.shape[1])
D1 = (D1 < keep_prob).astype(int)
A1 = A1 * D1
A1 = A1 / keep_prob
```

We should shut off same hidden units when we compute back propagation and also scale derivatives by keep_prob.

![](../../../../images/Pasted%20image%2020240129143948.png)

Things to remember :
- use dropout only in training
- apply dropout both forward and backward propagation
- To keep same expected value, we should divide activation by keep_prob