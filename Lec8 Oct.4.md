```c++
#include "stdio.h" // look into local dir
#include <vec.h> // look into system
```



```c++
// vec.h
struct Vec {
  int x,y; // declar
};
```

```c++
// vec.cc
#include <vec.h>
...
```

```c++
// main.cc
#include <vec.h>
...
```

Issue: wont compile

- vec.cc - missing main
- main.cc - missing vec implementation 



Processer -> compiler -> linker

- pass through src code
- compiler: compile each independently 
- linker: puts all together

`g++14 -c main.cc // main.o` 

`g++14 -c vec.cc // vec.o`

`g++14 main.o vec.o -o main // exe` 

vec.h? - do not compile this



## Global Variable

eg. variable 

```c++
// intuitive 
// abc.h
int globalNum; // decl + definition
```

- cannot just include in header - source files each get their own def'n 

```c++
// abc.h 
extern int globalNum; // declaration only (defined elseware)
```

```c++
// main.cc
#include <abc.h>
int main() {
  int globalNum = 5; // definition - one source file only
}
```



eg. linear algebra module using vector class

BAD

```c++
// linalg.h
#include <vec.h>
...
```

```c++
// linalg.cc
#include <linalg.h>
#include <vec.h>
```

```c++
// main.cc
#include <vec.h>
#include <linalg.h>
int main() {
  ...
  Vec v;
  ...
}
```

- this will not compile 
  - main.cc, vec.cc each gets multiple copies of vec.h
  - problem ? struct (Vec) is defined multiple times - BAD
- issue -need to prevent headers from being included multiple times
- sol'n 1 - be careful -- i.e manually track dependencies -- nearly impossible 
- sol'n 2 - header guard / include guard

eg. include guard for vector

```c++
// vec.h
#ifndef _VEC_H_
#define _VEC_H_
struct Vec {
  int x,y;
}
#endif
```

guidelines

- always use an include guard in header files
- **never,ever** put `using namespace std` in a header file



# Classes

- "big innovation" in OOP
- idea that structs can also have functions(behavior) 
  - data + behavior (in a struct)



def'ns 

- class 
  -  struct type that can potentially contain functions 
- object 
  - an instance of a class
  - **member variables **- instances of variable for an object 
  - **member functions **- functions inside a class



eg. 

```c++
struct Student {
  int assns, mt, final_grade;
  float grade() {
    return 0.4 * assns + 0.2 * mt + 0.4 * final_grade;
  }
};
```



- grade() is a function that can be called 
- assns, mt, final are member variables that hold values for an instance of student
- `Student billy {70,80,22}; // assns,mt,final`
- `cout << billy.grade(); // uses Billy's data`

Formally:

- methods take an extra "hidden" parameter, called ***this*** which is a pointer to the instance of a class
- can rewrite like this:

```c++
struct Student {
  int ...;
  float grade() {
    return 0.4 * this->assns +
    	0.2 * this->mt +
    	0.4 * this->final_grade;
  }
};
```



Initializing objects 

```c++
Studentt billy {70,80,90};
// numbers are assigned ti data members in order
// 70 -> assns
// 80 -> mt
// 90 -> final
```



define a method for initialization - constructor 

```c++
struct Student {
  int assns, mt, final_grade; // same
  float grade() {...}
  Student (int assns, int mt, int final_grade) {
    this->assns = assns;
    this->mt = mt;
    this->final_grade = final_grade;
  }
};

Student billy {60,40,12}; // calls constructor for int
```

- if a constructor is defined, this calls the constructor
- if a constructor is not defined, direct initialization



### uniform initialization 

- different forms of initialization, same effect

```c++
int x = 5;
string s = "hello";

int x {5};
string s {"hello"};
Student jane {95,85,98};
```



advantage of constructors

- default parameters can be applied/used
- overloading 
- sanity checks
- logic

default constructor

- if you dont define any constructors the compiler will create one for you
- default - no argument constructor 
  - eg. Student() i.e no argument passed
- what does default do?
  - default constructs objects 
  - get the effects to direct init.

```c++
sturct Student {
  int assns, mt, fi;
  float grade() {...}
}; // no constructor, just default
Student rena; // struct allocate call default constructor
Student tennie {10, 20, 30}; // default, initializes
```



