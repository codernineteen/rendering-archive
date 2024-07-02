
When we create a global instance in c++, we need to add a proper keyword in front of the declaration.
Otherwise, it will create duplicate symbols errors following the characteristics of linkage steps on cpp project.

1. With static
A static in global context means the symbol has internal linkage, which is only visible in the cpp file in which it lives.
```cpp
// Under contents in Global.h
#pragma once

namespace Global {
	struct Thing
	{
		int X = 0;
	}

	static Thing thing;
}

//...
// In another file 
// main.cpp

#include "Global.h"

static Global::Thing thing; // this is a separate copy of original instance.
```

2. with static in certain class
This guarantees that the obtained object of 'thing' from Global file is the same object with the its origin.
```cpp
#pragma once

namespace Global {
	struct Thing
	{
		int X = 0;
	}

	class Things
	{
	public:
		inline static Thing thing;
	}
}
```

3. with inline keyword
inline is a kind of magical keyword. It ensures that there is always one instance of 'thing' regardless of how many we include the file in somewhere else.
```cpp
#pragma once

namespace Global {
	struct Thing
	{
		int X = 0;
	}

	inline Thing thing;
}
```


4. With extern keyword.
By telling the class is defined in another file to the class with extern keyword, we can declare a certain instance without internal linkage.
```cpp
#pragma once

extern class MKCommandService* GCommandService;
```

Then by defining another class which initializes every global instances in its constructor in cpp file, we can use the global instances by including header file.
```cpp
#include "Global.h"
#include "MKCommandService.h"

MKCommandService* GCommandService = nullptr;

class MKGlobal
{
public:
	MKGlobal()
	{
		GCommandService = new MKCommandService();
	}

	~MKGlobal()
	{
		delete GCommandService;
	}
} GlobalInstance; // call the MKGlobal constructor when the program start.
```
Because the lifetime scope of this 'GlobalInstance' is the outer-most of the program, we will delete dynamically allocated global instances when the program ends.