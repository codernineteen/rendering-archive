
# Key things to remember

- preprocess data set
	- figure out the dimensions and shaped of the problem
	- reshape the datasets appropriately
	- **Standardize** the data
- Logistic regression model overview (mathematical representation)
- ![[Pasted image 20240106155153.png]]
- Initialize(w, b)
- Optimize the loss iteratively to learn parameters (w,b)
	- compute the cost and its gradient
	- update the parameters using gradient descent
- Use the learned (w,b) params to predict the labels for a give set of examples

# Make a Model
1. Pre-process the dataset is important
2. Modularize a model by implementing separate functions
	- initialize parameter
	- propagate function to compute 'cost' and 'derivatives'
	- optimize function to update parameters
	- predict fuction
