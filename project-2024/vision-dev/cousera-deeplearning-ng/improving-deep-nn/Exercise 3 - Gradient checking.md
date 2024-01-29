
Goal : Prove if our back propagation actually works.

Using two-sided difference,
$$ \frac{\partial J}{\partial \theta} = \lim_{\varepsilon \to 0} \frac{J(\theta + \varepsilon) - J(\theta - \varepsilon)}{2 \varepsilon} \tag{1}$$


# 1-D gradient check

![](../../../../images/Pasted%20image%2020240129185415.png)
- get a cost from forward propagtation
- get a true gradient by our backward propagation implementation
- compare an approximation of gradient with true gradient by following :
 $$ difference = \frac {\mid\mid grad - gradapprox \mid\mid_2}{\mid\mid grad \mid\mid_2 + \mid\mid gradapprox \mid\mid_2} \tag{2}$$
If this difference is under 2 x epsilon, we have a valid implementation

# Multi dimensional gradient check
In our neural network,
- cost is determined by all of parameters $(W_1, b_1, .... W_l, b_l)$
- We have each gradients for every parameters.

The core of gradient checking in multi-dimenson is quite similar to 1-D way, But the difference is that we need to repeat it for each of parameters by getting every gradient approximations of them.
![](../../../../images/Pasted%20image%2020240129185732.png)

**What we should remember from this notebook**:
- Gradient checking verifies closeness between the gradients from backpropagation and the numerical approximation of the gradient (computed using forward propagation).
- Gradient checking is slow, so you don't want to run it in every iteration of training. You would usually run it only to make sure your code is correct, then turn it off and use backprop for the actual learning process

# Reference 
https://www.coursera.org/learn/deep-neural-network/programming/RS2w3/gradient-checking