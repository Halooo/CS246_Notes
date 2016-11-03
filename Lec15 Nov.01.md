### Destructor (revisited) 

Bad code

```c++
class x {
  int *x;
  public:
  	x(int n): x{new int [n]} {};
    ~x() {delete [] x;}
};

class y: public x {
  int *y;
  public:
    y(int m, int n):x{n}, y{new int []} {}
    ~y(){delete [] y;}
};

X *myX = new y {10, 20};
delete myX;
```

- what is the problem with this code??
  - This leaks memory
  - b/c only `~x` destructor for x runs, that for y never runs
- Q: How do we ensure that the subclass destructor runs, even when using a Base class pointer?
  - A: Declare the destructor virtual - forces the compiler to actually determine the type at runtime

```c++
class x {
  ...
  public:
    ...
    virtual ~x() {delete [] x;}
};
```

Rule -

- always make the destructor virtual in classes that are meant to be subclasses
  - even if it does nothing! b/c the subclass's destructor might do something 
- If you don't want a class to be subclassed, make it `final`

eg: 

```c++
class y final : public x { // ie. yea cannot subclass y
  ...
};
```



## Pure Virtual Methods (Abstract Classes)

eg

``` c++
class Student{
protected:
  int num Classes;
public:
  vitual int fees(); // calculate fees
}

class Regular: public student {
public:
  int fees() override; // calc. regular student fees
};

class Coop: public Student {
public:
  int fees() override; // calc coop student fees
}
```

Q: re Student impl
- what do I do with `Student::fees()` ?
  - I'd like this method to have no implementation ..
  - make it a **pure virtual** method

eg: 
``` c++
class Student {
public:
  virtual int fees() = 0;
}
```

Once you add a pure virtual method to a class, that class canot be instantiated
ie
``` c++
Student s; // error - can only instantiate Regular or Coop
```

- such a class is called an **Abstract Class** 
  - purpose is to serve as a base lass / organize subclasses
  - subclasses of abstract classes are also abstract unless you implement all pure virtual methods
  - classes that can be instantiated (incl. subclasses implementing pure virtual methods) are **Concrete Classes**


**in UML:**
- represent virtual and pure virtual methods in ***italics***
- represant abstract classes with name in ***italics***
  eg:

*Student*
|__Regular
|__Coop



## Templates

recall:

``` c++
class List {
  struct Node;
  Node * theList;
};

struct List::Node {
  int data;
  Node *next;
}
```



Q: what if I want a List that supports something other than int?

A1: copy/paste code into a new class ...

- not scalable / maintainable

A2: to use \<Template\> class 

- A template class is a class that is parameterized by **type**

eg: Stack

``` c++
template <typename T> class Stack {
  int size;
  int cap;
  T * contents;
public:
  Stack() {...}
  void push (T x) {...}
  T top() const {...}
  void pop() {...}
};
```

We can rewrite our List class ad a template :

``` c++
template <typename T> class List {
  struct Node {
    T data;
    Node *next;
  };
public:
  class Iterater {
    Node *p;
    explicit Iterator (Node *p):p{p} {}
  public:
    T &operator*(){...}
    friend class List<T>;
  };
  T ith(int i) {...}
  void addToFront(const T &n) {...}
};
```

possible client code

``` c++
List <int> l1;
List <list<int>> l2;
l1.addToFront(s);
l2.addToFront(l1);
```

``` c++
for (auto it: li) {
  cout << it << endl;
}
```



Compiler generates code for templates, loosed on usage. 



## Standard Template Library (STL)

- originally, this was a set of external libraries for working with templates
  - ie. use templates to add features to c++
  - now included in c++
- eg. Vector - dynamic length array

eg: Vectors

```c++
#include <vector>
using namespace std;
vector <int> v {4,5}; // init with values 4,5
v.emplace_back(6); // 4,5,6
v.emplace(7); // 7,4,5,6
```

- methods to insert, erase...



loop over vectors:

```c++
for (int i = 0; i < v.size(); ++i) {
  cout << v[i] << endl;
}
```

Iterator abstraction:

```c++
for (vector<int>::iterator it = v.begin(); it != end(); ++it) {
  cout << *it << endl;
}

// or

for (auto n : v) {
  cout << n << endl;
}
```



To iterate in reverse:

``` c++
for (vector<int>::reverse_iterator it = v.rbegin(); it != rend(); ++it) {
  ...
}

// or

for (auto it = v.rbegin(); it != v.rent(); ++it) {
  ...
}

v.pop_bacl(); // removes las element
```



Advantage?

- easy to work with - no explicit new/delete to manage it
- vectors are guaranteed to be implemented as arrays
- use whenever you need dynamic - length array
  - avoid vector versions of new and delete

Other vector operators work with iterators - eg erase

``` c++
auto it = v.erase(v.begin()); // erases item 0, return iterator to next item
it = v.erase(v.begin() + 3); // erases 4th item
it = v.erase(it); // erases item pointed to by iterator
it = v.erase(v.end()-1); // erase last item in list
```



```c++
v[i]; // returns ith element - unchecked - 
// eg vector sizes; v[10] 
// fails with unspecified errr(undefined behaviour)

v.at(i); // returns ith element - checked
// compiler has predictable behaviour
```



# Exceptions

What do you do, when errors occur?

Problem:

- vector code (for `v.at[i]`) can detect the errors, but does it know what to do with it?
- the client/program knows what to do, but cannot directly detect error
  - detect errors: local problem "class/function"
  - react to errors: global problem "client"

How does vector tell the program that there's an error?

- **C sol'n**: 
  - return status code from a function
  - or set global value like an `int error`
  - these are not great, encourage programmers to skip errors
- **C++ sol'n**: when an error arises, the function **raises an exception**
  1. (exception is a type of error), by default, the program halts
  2. we can write a handler to **catch the exception**

eg: `Vector <T>::at` raises an exception `std::out_of_range`

``` c++
#include <stdexcept>
try {
  cout << v.at(1000000) << endl;
}
catch (out_of_range) {
  cerr << "Range Error" << endl;
}
```

