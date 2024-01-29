
A well-chosen initialization can:

- Speed up the convergence of gradient descent
- Increase the odds of gradient descent converging to a lower training (and generalization) error


# 1. zero initialization
```python
parameters['W' + str(l)] = np.zeros((layers_dims[l], layers_dims[l-1]))
parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))
```

relation between cost and iterations.
![](../../../../images/Pasted%20image%2020240129112319.png)
- performance is worst
- no better than random guessing( based on 0.5)

With this zero initialization,
- every ReLU activation follows
$$a = ReLU(z) = max(0, z) = 0$$
- a sigmoid activation for output :
$$\sigma(z) = \frac{1}{ 1 + e^{-(z)}} = \frac{1}{2} = y_{pred}$$

And our loss function will be :
$$ \mathcal{L}(a, y) =  - y  \ln(y_{pred}) - (1-y)  \ln(1-y_{pred})$$
when y = 1,
$$ \mathcal{L}(0, 1) =  - (1)  \ln(\frac{1}{2}) = 0.6931471805599453$$
when y = 0,
$$ \mathcal{L}(0, 0) =  - (1)  \ln(\frac{1}{2}) = 0.6931471805599453$$

In conclusion, neurons learn same thing and this leads to the network having same power with a linear classifier.

# 2. Random initialization
"Break symmetry"

This random initialization breaks symmetry on weight matrices, but it has problems of vanishing or exploding weights.

- ex) initialze large matrices multiplying by 10
```python
parameters['W' + str(l)] = np.random.randn(layers_dims[l], layers_dims[l-1]) * 10
parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))
```

- the result of cost
 ![](../../../../images/Pasted%20image%2020240129113552.png)

We can see that cost is really high at the beginning of learning.
-> Poor initialization can lead to vanishing / exploding gradients
-> these problems slow down our optimization algorithm

Note that 
- numpy.random.rand() - uniform distribution
- numpy.random.randn() - normal distribution
And randn() helps most weight to avoid begin close to extremes.
-> Remind that our learning become slow when we have very high/small weights that leads to high/small 'z' in sigmoid funciton.

# He initialization
- weight scaling factor - $\sqrt{\frac{2}{dimension(l-1)}}$
```python
parameters['W' + str(l)] = np.random.randn(layers_dims[l], layers_dims[l-1]) * np.sqrt(2.0/layers_dims[l-1])
parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))
```
![](../../../../images/Pasted%20image%2020240129114450.png)

Accuracy is the best when the parameters is initialized in a 'He initialization' way

# Reference

- https://www.coursera.org/learn/deep-neural-network