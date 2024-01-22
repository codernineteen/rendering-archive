
# Setting up ML application

 we need to determine :
 - number of layers
 - number of hidden units
 - learning rates
 - activation functions

# Datasets in ML
- Training set : train a model to predict a label based on train inputs
- Development set : cross validation
- Test set : test actual result by comparing prediction with true label
![](../../../../Pasted%20image%2020240122123846.png)

Setting up ML application
-> set it up into a train, dev and test sets.

- If we have relatively small datasets, it might be okay to follow tranditional partitioning datasets rule, which is "7:3"  or "6:2:2" in general.
- If we have a big data set, it is okay to use much smaller dev and test sets.

A few advices from professor Ng
- **Make sure that dev and test sets come from same distribution.**
	- this may leads to faster learning algorithm
- It might be okay not to have test sets if we don't need unbiased estimate.

``