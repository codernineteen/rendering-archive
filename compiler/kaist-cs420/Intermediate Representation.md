# CFG (control flow graph)

statement based codes -> control flow graph conversion.
# register machine
- IR only handles operation in the level of registers, not memory.

One of the advantages of using register is 'non-interference'
With memory, it's possible to do below
```c++
var = 42;
foo(&var);
assert(var == 42); // may fail due to call 'foo'
```

On the other hand, a register cannot be address-taken
```
 %var = 42;
 // foo(&%var) <- impossible
 assert(%var == 42); // always true
```


# Unambiguous variable names

- In code level, there are often ambiguity in variable names.
```c++
int i = 0;
int s = 0;
for(int i = 0; i < 10; i++) s++; // i here is different variable with outer scope

cout << i << endl; // result : 0
```

Intermediate representation resolve this ambiguity by renaming variables.
So it is our duty to implement such a name resolver in IR generation unit.

# Scoping

- dynamic scoping : resolution depends on the execution (such as interpreter languages.). This kind of languages are not modular.
- static scoping : resolution depends solely on the AST. (such as compile languages.). This kind of languages are modular.


# Explicitly annotated types

Motivation : it is neccessary for generating asembly-level instructions.

In C/C++, the code doesn't always annotate types.
But IR always annotate type explicitly.
```
// in c, 42 is converted into float32. Then, do add operation.
42 + 0.4f

// in IR
(42 as int32) + (0.4f as float32)
```


# Static single assignment(SSA)\

a register is defined at most once in code. (but it may be assigned multiple times in different execution context such as for-loop)
```
%var = 42;
...
%var = 777; // impossible.
```

-> This leads code opitmization to be much easier. 