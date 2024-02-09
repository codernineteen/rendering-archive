
# 1. auto

- type deduction using auto keyword 
	-> reducing manual spelling long type name
	-> flexibility on type change.

## 1.1 template type deduction

ex)
```cpp
template<typename T>
void f(ParamType param);
...
f(expr);
```
-> compiler use 'expr' to deduce types of 'T' and 'ParamType'


Three cases for ParamType :
### a pointer or reference, but not universal reference (simplest)
```cpp
template<typename T>
void f(T& param);

int x = 27;
const int cx = x;
const int& rx = &x;
f(x); // T - int, param type - int&
f(cx);// T - const int, param type - const int&
f(rx);// T - const int, param type - const int&
```
Note that T is deduced to non-reference in `f(rx)` call.

If we add const keyword in ParamType, there is no need to deduce 'const' in T.
```cpp
template<typename T> // always deduce without 'const'
void f(const T& param);
```

Pointer works perfectly in a same way.


###  a universal reference


### neither a pointer nor a reference