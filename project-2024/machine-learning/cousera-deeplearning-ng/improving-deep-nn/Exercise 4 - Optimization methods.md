
# Goal

- Apply optimization methods such as (Stochastic) Gradient Descent, Momentum, RMSProp and Adam
- Use random minibatches to accelerate convergence and improve optimization


# 1.  gradient descent vs Stochastic gradient descent

![](../../../../images/Pasted%20image%2020240203143537.png)![](../../../../images/Pasted%20image%2020240203143701.png)

# 2. Mini-batch gradient descent
1. shuffle
   Shuffle X and Y synchronously. (the location of each training example in shuffle version is dependendent on each other.)
![](../../../../images/Pasted%20image%2020240203143900.png)

2. Partition (following power of 2)
The size of final mini batch might be not power of 2 , but it is okay.
The final mini-batch size follows a formulat - $\left(m-mini_\_batch_\_size \times \left\lfloor \frac{m}{mini\_batch\_size}\right\rfloor\right)$. 
![](../../../../images/Pasted%20image%2020240203144604.png)

- python code for mini-batch size
 For complete size of mini-batches (k is current mini-batch)
```python
mini_batch_X = shuffled_X[:, k * mini_batch_size : (k+1) * mini_batch_size]        
        mini_batch_Y = shuffled_Y[:, k * mini_batch_size : (k+1) * mini_batch_size]
```

For imcomplete last mini-batch, (if it exists)
```python
mini_batch_X = shuffled_X[:, num_complete_minibatches * mini_batch_size : m]
mini_batch_Y = shuffled_Y[:, num_complete_minibatches * mini_batch_size : m]
```

**What we should remember**:

- Shuffling and Partitioning are the two steps required to build mini-batches
- Powers of two are often chosen to be the mini-batch size, e.g., 16, 32, 64, 128.

# 3. Momentum
We calculate exponentailly weighted averages, which is velocity.
This make our model keep the momentum based on previous direction.

**What we should remember**:
- Momentum takes past gradients into account to smooth out the steps of gradient descent. It can be applied with batch gradient descent, mini-batch gradient descent or stochastic gradient descent.
- You have to tune a momentum hyperparameter 𝛽 and a learning rate 𝛼.

# 4. Adam
One of the most effective optimization algorihtms for training neural networks.
$$\begin{cases}
v_{dW^{[l]}} = \beta_1 v_{dW^{[l]}} + (1 - \beta_1) \frac{\partial \mathcal{J} }{ \partial W^{[l]} } \\
v^{corrected}_{dW^{[l]}} = \frac{v_{dW^{[l]}}}{1 - (\beta_1)^t} \\
s_{dW^{[l]}} = \beta_2 s_{dW^{[l]}} + (1 - \beta_2) (\frac{\partial \mathcal{J} }{\partial W^{[l]} })^2 \\
s^{corrected}_{dW^{[l]}} = \frac{s_{dW^{[l]}}}{1 - (\beta_2)^t} \\
W^{[l]} = W^{[l]} - \alpha \frac{v^{corrected}_{dW^{[l]}}}{\sqrt{s^{corrected}_{dW^{[l]}}} + \varepsilon}
\end{cases}$$






# Reference
https://www.coursera.org/learn/deep-neural-networ