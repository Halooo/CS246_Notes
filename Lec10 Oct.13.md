Recap:

Destructors 

- method called destructor runs when an object is destroyed 
  - stack allocated - out of scope
  - heap allocated - `delete`
- form 

``` c++
~ Student() {
  ...
}
```

- sequence 
  1. dtor called for object
  2. dtor called for fields that are objects 
  3. space reclaimed
- default dtor - just invokes dotrs for fields that are objects 
- why do you need to write own ones? - ***dynamic memory***



eg

``` c++
void foo() {
  Node *np = new Node; // stack - np will be destroyed b/c on stack
  ... // anything in heap will leak memory
}
 void bar() {
   Node * np2 = new Node; // np2 - stack
   ...
   delete np2; // leaks memory  the first Node is freed but others linked to it is not
 }
```



``` c++
struct Node {
  ~ Node() {
    delete next; // dtor
  }
  int data;
  Node *next; 
}

Node *np = new Node;
...
delete np;
```



## Copy Assignment Operator

``` c++
Student billy {30,45,50}; // ctor
Student jane = billy; // copy ctor - initializing from a copy
Student joey; // ctor
joey = billy; // copy assignment - after initialization
```

- classes have a default copy assignment operator
  - does a shallow copy
- when do you need to write your own copy-assign operator?
  - dynamic memory

``` c++
struct Node { // attempt #1
  ...
  Node &operator = (const Node &other) {
    data = other.data;
    delete next;
    next = other.next ? new Node{*other.next} : nullptr;
    return *this;
  }
};
```

- why is this problematic? - what if we do this: 

``` c++
Node n;
n = n; // self-referencial comparisions
	   // end up deleting this.next other.next
	   // bad - deleted data
```



While writing operator, always check for self-assignment

``` c++
struct Node {
  int data;
  Node *next;
  Node &operator= (const Node &other) { // attempt #2 - fixes self-assignment
    if (this == &other) return *this;
    data = other.data;
    delete next;
    next = other.next ? new Node{*other.next} : nullptr;
    return *this;
  }
};
```



other issues?

1. what if `other` points to a node in my list?
   - can inadvertently delete other's list
2. what if new fails? we have freed old data before new node is created - data loss
   - data loss
   - ideally, delete old data **last** 



``` c++
struct Node {
  ...
  Node & operator = (const Node &other) { // attempt #3
    if (this == &other) return *this;
    Node *tmp = next;
    next = other.next ? new Node{*other.next} : nullptr; // OK. if new fails here, function will return
    data = other.data;
    delete tmp;
    return *this;
  }
};
```



- copy & swap idiom

``` c++
#include <utility>
struct Node {
  ...
  void swap(Node &other) {
    using std::swap;
    swap(data, other.data);
    swap(next, other.next);
  }
  Node &operator= (const Node &other) {
    Node tmp = other; // copy ctor
    swap(tmp); // swap with tmp
    return *this; // tmp stack allocated - when it goes out of scope, tmp is destroyed - and old data with it
  }
};
```



## R-value & R-value References 

`int x = 5` 

`int x` l-value have an address 

`5` r-value anything not an l-value



consider 

```c++
Node n{1, new Node{2,nullptr}};
Node m = n; 
Node m2;
m2 = n; // copy assign

Node plusOne(Node n) {
  for (Node *p = &n; p; p = p->next) {
    ++p->data;
  }
  return n;
}
Node m3 = plusOne(n); // copy ctor
```



- compiler creates temporary objects
  - ie plusOne) returns a temp
  - this is just going to get deleted as soon as the statement ends
- it is *wasteful* to copy from a temporary since we are just going to discard it
  - instead of copying the data from a temp 
- **But** I have to know that my r-value is a temp to steal from it



How do I tell if I have a temp object?

- in c++, an r-value reference (&&) refers to a temp object

eg: Node && - reference to a tmp r-value of type Node

Node & - l-value reference



We can write a version of copy ctor that works with - rvalues (temp object)

``` c++
struct Node{
  ...
  Node (Node && ): data{other.data}, next{other.data} {
    other.next = nullptr;
  }
}
```

- so, a **move constructor** steals data from an r-value
- Similarly

```c++
Node n;
m = addOne(n); // temporary object - used by copy assign operator
			  // costly - copy ctor being called, then tmp discarded
			  // steal it instead!
```



- to avoid copy/assign from temp, create a **move assignment operator**

```c++
struct Node {
  ...
  Node & operator = (Node && other) {
    using std::swap;
    swap(data, other.data); // no temp
    swap(next, other.next);
    return *this;
  }
};
// when returns, other is destroyed (tmp) and takes original data
```



- if you don't define a move ctor/move assignment operator, get copy versions instead
- if a move ctor/move assign op is defind, it will replace copy ctor/assignment for r-value references (temps)

