# lvalue vs rvalue
- lvalue : 
	- a value whose address is gettable.
	- copies of lvalues are generally copy constructed.
- rvalue : 
	- we can't take an address of value.
	- copies of rvalues are generally move constructed.

# argument vs parameter
- argument : a value passed during function invocation (rvalues in general)
- parameter : a variable in a function definition (placeholder)