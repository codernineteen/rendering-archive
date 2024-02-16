
# 1. What is constant expression?

If a compiler can determine the value of certain expression, the expression is called 'Constant expresssion'.
If the type of value of constant expression is integer type, then it is called 'Integral constant expression'.

- examples of integral constant expression
1. size of static array
```cpp
int arr[size]; // 'size' should be integral constant expression
```
2. enum variant's value designation
```cpp
enum Animal { CAT = number, DOG, TIGER }; // number should be integral constant expression.
```

# 2. constexpr keyword
`constexpr` keyword specify an expression as constant.
```cpp
int main() {
	constexpr int size = 3; // now size is constant expression
	int arr[size]; // valid expression
}
```


# 3. constexpr vs const
- const
```cpp
int a;
const int b = a;
```

- constexpr
```cpp
int a;
constexpr int b = a; // ?
```

The difference is that constexpr requires constant expression at the right side of the expression.
So `constexpr` is always `const`, but `const` is not always `constexpr`.

In conclusion, if we really want to use constant at compile time certainly, we should use `constexpr` instead of `const`


# 3. constexpr application
If we add a `constexpr` keyword in front of return type of a function, we can make the return value of function as compile-time constant.
-> because we already determine the return value at compile time, we can reduce runtime overhead.

```cpp
constexpr int factorial(int N) {
	int ret = 1;
	for(int i = 1; i < N; i++)
		ret *= i;
	return ret;
}

template <int N> // requires integral constant expression.
struct A {
	int operator()() { return N; }
}

int main {
	A<factorial(10)> a; // this is valid at compile!
}
```

If we pass compile time constants as function parameters, the return value also becomes constant at compile time.
On the other hand, we still can pass non-constant values into parameters. In this case, the function will work as normal runtime function.
```cpp
constexpr int con = 10;
int nonCon = 10;
A<factorial(con)> a; // constant
factorial(nonCon); // normal function 
```


# 4. constraints of constexpr
 There are some constraints that we can't use `constexpr`
- goto statement
- exception handling(but c++20 possible)
- non-literal type
	Literal type is a type which cpp compiler can define the type of variable at compile time.
	Here are some literal types:
		- `void`
		- scalar type (`int`, `char`, `bool`, ....)
		- reference type
		- array of literal types
		- a type satisfy below conditions
			- has default destructor
			- And corresponding to one of below cases
				- lambda function
				- Aggregate type(ex. pair)
				- has `constexpr` constructor and no copy & move constructor
- uninitialized variable definition
- when the expression included a function call which is not `constexpr`
	```cpp
	void nonConstFunc() { ... }
	constexpr int constantFunc(int n) {
		//Do something
		nonConstFunc(); // this causes error.
		return n;
	}	
	```

# 5. constexpr constructor
Thanks to expansion of literal types, now we can make some user-defined literal types.

ex) a literal class
```cpp
class Dog
{
public:
	constexpr Dog(int age, char gender) : _age(age), _gender(gender) {}

	constexpr int age() const { return _age; }
	constexpr char gender() const { return _gender; }

private:
	int _age;
	char _gender;
}

int main() {
	constexpr Dog d1{1, 'M'}; // we can declare an instance as constant
	// because we initialized it with constexpr constructor.
}
```


# 6. if constexpr
What if we want to define a function that works differently according to given parameters?
With `constexpr`, we can do this more efficiently.

ex) 
without `if constexpr`, 
```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void show_value(T t) {
  if (std::is_pointer<T>::value) {
    std::cout << "This is pointer: " << *t << std::endl;
  } else {
    std::cout << "This is not pointer : " << t << std::endl;
  }
}
```
This template function will be instantiated with a call `int x = 10; show_value(x);` like below.
```cpp
void show_value(int t) {
  if (std::is_pointer<int>::value) {
    std::cout << "This is pointer: " << *t << std::endl;
  } else {
    std::cout << "This is not pointer : " << t << std::endl;
  }
}
```
Even though if statment never be true, compiler complains about dereferencing non-pointer type is invalid behavior.

We can solve this problem by adding `constexpr` in conditional statment.
```cpp
template<typename T>
void show_value(T t) {
  if constexpr (std::is_pointer<T>::value) {
    std::cout << "This is pointer: " << *t << std::endl;
  } else {
    std::cout << "This is not pointer : " << t << std::endl;
  }
}
```
With `constexpr` keyword, There are two possible cases.
- If `if constexpr` is true, `else` scope will **never be compiled**.
- If `else` is true, `if constexpr` scope will never be compiled.
In above instantiated case, `int` is not a pointer type. So the code inside `if constexpr` will not be compiled and there will be no error.
Notice that `if constexpr` should be constant expression which can be converted into `bool` type at a compile time.

# Reference
- https://modoocode.com/293