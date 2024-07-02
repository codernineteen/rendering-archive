
- encapsulate each resource into a class.
	- The class's constructor acquires the resource and establishes all class invariants or throws an exception if that cannot be done
	- The destructor releases the resource and never throws exceptions
- always use the resource via an instance of **RAII-class**
	- the class has automatic storage duration or temporary lifetime itself.
	- the class has lifetime that is bounded by the lifetime of an automatic or temporary object

```cpp
std::mutex m;
 
void bad() 
{
    m.lock();             // acquire the mutex
    f();                  // if f() throws an exception, the mutex is never released
    if (!everything_ok())
        return;           // early return, the mutex is never released
    m.unlock();           // if bad() reaches this statement, the mutex is released
}
 
void good()
{
    [std::lock_guard](http://en.cppreference.com/w/cpp/thread/lock_guard)<[std::mutex](http://en.cppreference.com/w/cpp/thread/mutex)> lk(m); // RAII class: mutex acquisition is initialization
    f();                               // if f() throws an exception, the mutex is released
    if (!everything_ok())
        return;                        // early return, the mutex is released
}
```


# Reference
https://en.cppreference.com/w/cpp/language/raii