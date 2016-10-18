### Constructor 

destructor *

copy constructor * , move constructor * (specialized copy constructor),

copy assignment operator * , move assignment operator * (specialized copy assignment operator)

### "Rule of 5"

If you need a custom version of any of *, then you usually need a custom version of ***all 5*** above



# Member Operators 

recall: copy assignment (operator = ) is a member fn 

eg:

```c++
struct Vec {
  ...
  vec operator = (const Vec &v); // member of Vec only has RHS 
};
```

- when an operator is declared as a member function, "this takes the place of the LHS argument"

``` c++
struct Vec {
  int x, y;
  ...
  vec operator + (const Vec &other) {
    return {x+other.x, y+other.y};
  }; // member of Vec only has RHS 
  Vec operator * (const int k) {
    return {k*x, k*y};
  }
  Vec operator * (const int k, const Vec &other) {
    return {k*other.x, k*other.y};
  }
};

Vec v1;
Vec v2;
Vec v3 = v1 + v2; // operator +
Vec v4 = v3 * 55; // operator *
Vec v5 = 2 * v4; // scalar LHS. Vec RHS


```



### I/O Operators?

```c++
struct Vec {
  ...
  ostream & operator << (ostream &out) {
    return out << x << " " << y;
  }
};
```



So, operator << and operator >> are standalone so that we have the correct operators

cin and cout are LHS, class RHS

- but there are some operators that **must** be member functions:
  - operator `= `
  - operator `[]`
  - operator `->`
  - operator `()`
  - operator `T` (where T is a type)
- side note: Separate Compilation of classes

```c++
// node.h
#ifndef _NODE.H_
#define _NODE.H_
struct Node {
  int data;
  Node * next;
  explicit Node (int data, Node *next = nullptr);
 
  Node (const while &n);
}
#endif
```



```c++
// Node.cc
#include <node.h>
Node::Node(int data, Node *next):data{data}, noext{next} {
  
}
Node::Node(const Node &n): data{n.data}, next {n.next ? new Node {*n.next} : nullptr} {
  
}
// :: scope resolution operator means "in the context of" eg Node method of class Node
```



## constant objects

recall:

```c++
int f (const Node &n)
```

- often pass `const` objects as parameters
- `const` indicate that its fields cant be modified
  - what about member functions?
    - risk of member function changing data (violating const)
- you can call functions on `const` objects if the methods promise not to change data/fields.

```c++
struct Student {
  int assns, mt, final; 
  float grade () const; // will not change data;
}
```

- compiler checks that we only call const methods of `const` object 
- but what if I am profing / debugging 

``` c++
struct Student {
  int numMethodCalls = 0;
  float grade() {
    ++nameMethodCalls; // nothing to do with structed data
    return...
  }
}
```

- grade cannot easily be const - can't increment `numMethodCalls()`
- but `numMethodCalss()` only violates physical const (ie. changes bits n memory, but doesn't affect grade)
  - doesnt affect logical constness (grade validity)
- so what to do?
  - we can make the field mutable, can be changed oven if object is const

```c++
struct student {
  ...
  mutable int numMethodCalls = 0;
  float grade() const {
    ++numMethodCalls; // const does not apply to mutable fields
  }
};
```



## Static Members

- numMethodCalls - applies to a single object
- what if I wont to count method calls for the class?
  - or track number of students created?
- **Static members** apply to the class, not the object (and not to any one instance of the class)

```c++
struct Student {
  static int numInstances;
  Student (int assns, int mt, int final):assns{assns}, mt{mt}, final{final} {
    ++ numInstances;
  }
};
int Student::numInstances = 0; // in the .cc file outside of scope of class
```



## Static Member Functions

don't depend on an object either for their existence 

- don't have access to a this pointer
- don't have access to fields
- can access static members



``` c++
// student.cc
struct Student {
  static int numInstances;
  Student (int assns, int mt, int final):assns{assns}, mt{mt}, final{final} {
    ++numInstances;
  }
  static void printInstances() {
    cout << numInstances << endl;
  }
};

int Student::numInstances = 0; 
// initialize the static variable outside of class b/c we do not want to reset the value of numInstances to 0 every time we construct it
```

```c++
// main.cc
int main() {
  Student::printInstances(); // 0
  Student billy {70,80,90};
  Student jane {80,85,90};
  Student::printInstances(); // 2
}
```



### Static Member Functions

1. dont rely o an instance of a class
2. don't have a "this" (ie. not necessarily have any instances of this class)
3. really just functions (not member functions)
4. can **only** access static members/fields
   -  converse not true, normal member functions can still access static members/fields



# Invariants . Encapsulation 

Consider: 

```c++
struct Node {
  int data;
  Node *next;
  Node (int data,Node *next): data{data}, next{next} {
    
  }
  ~Node() {delete next;}
};
```

```c++
int fun() {
  Node n1 {1, new Node {2, nullptr}};
  // n1 is in stack, pointing to a node in heap
  
  Node n2 {3, nullptr};
  Node n3 {4, &n2};
  // n2 and n3 are both in stack, n3 pointing to n2
  // if call delete n3 here, since n2 is not in heap, program is crashed
} // returns
```

- we assumed in the design that `next` was a ptr to the heap
- users did not know this!

**invariant ** - an assumption that must hold true for your program to operate correctly

ie - invariant in this case was that our pointer was either a valid ptr to heap(or nullptr)

- issue: cant generate this invariant (and cant generate correct usage by our user)