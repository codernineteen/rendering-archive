
# Goal

- Use `tf.Variable` to modify the state of a variable
- Explain the difference between a variable and a constant
- Train a Neural Network on a TensorFlow dataset


# Basics
```python
import tensorflow as tf
```

- tf.Tensor : equivalent of Numpy arrays.
- tf.Variable : store the state of our variables.
- tf.data.Dataset : tensorflow data generator (can only be accessed with iterator)
	- If we want to modify tensorflow dataset, we need to use method on the iterator because it is basically generator.
- tf.constant : store the state, but can't modify it unlike tf.Variable.

## examples of tf.Variable and tf.constant

- Parameter initialization
```python
    X = tf.constant(np.random.randn(3,1), name = "X")
    W = tf.Variable(np.random.randn(4,3), name = "W")
    b = tf.constant(np.random.randn(4,1), name = "b")
    Y = tf.add(tf.matmul(W, X), b)

```



# One-hot encoding
map a number in certain colume to a column vector that has the number at the corresponded component location.
![](../../../../Pasted%20image%2020240210175759.png) 

- one-hot encoding in tensorflow
```python
tf.one_hot(labels, depth, axis=0)
```
To reshape it as column-major matrix,
```python
one_hot = tf.reshape(tf.one_hot(label, C, axis=0), shape=[C,])
```

Now we can transform the original label vector into one-hot encoded matrix
```python
# a convert function named as 'one_hot_matrix'
new_y_test = orig_y_test.map(one_hot_matrix)
```

# Glorot initializer

- `tf.keras.initializers.GlorotNormal`
	- draws samples from a **truncated normal distribution centered on 0**
	- standard deviation $\sqrt{\frac{2}{fanIn + fanOut}}$
		, where fan-in is the number of input units and fan-out is number of output units

```python
initializer = tf.keras.initializers.GlorotNormal(seed=1) 
W1 = tf.Variable(initializer(shape=(25, 12288)))
```



# A neural network example in tensorflow

## forward propagation

- After initializing parameters, build a forward propagation structure
- Unlike a raw numpy , we use tensorflow built-in helper functions
	- tf.math.add
	- tf.linalg.matmul
	- tf.keras.activations.relu
## Compute loss
1. compute total loss (summing the losses from one-mini batch of samples)
	This prevent a case like :
		- mini-batch size : 4 and  we have 5 samples
		- mini-batch 1 : (x1, x2, x3, x4)
		- mini-batch 2 : (x5)
	-> If we used a mean loss for each mini-batch , the final cost will be different with desirable value.
1. Accumulate the sums from each of the mini-batches, and finishing it with division by the total number of samples -> get final cost value 