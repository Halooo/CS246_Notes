# Measures of Design Quality

Coupling and cohesion -at odds

**Coupling ** 

- the degree to which modules in a program depend on one another.
- (low) modules that communicate with basic function calls / parameter / return types (eg: ints, strings)
- (med) modules that pass data back and forth as structs / arrays
- (med) modules that affect each others control flow
- (med) modules sharing global data - all rely on common assumptions about data
- (high) modules that access each others implementations (eg. friends)

High coupleing?

- changes one module changes in another
- hard to reuse modules



**High coupling?**

- changing are module often requires changing alters
- hard to reuse



**Cohesion**

- how closely related elements of a module are to one another
- (low) arbitrary of customized elements/classes (eg: utility)
- (low) elements that share a theme, perhaps share some common code (eg: algorithm)
- (med) elements that manipulate state of something over a lifetime
- (med) elements that pass data to each other
- (high) elements that work together to perform a single task

**Low Cohesion?**

- partly organized code
- hard to understand / maintain



**Goals** 

- low coupling 
  - promotes reuse
- high cohesion
  - promotes readability/maintainability



# Decoupling the Interface (MVC)

As an example - most programs should not be printing to the screen ()

eg: 

``` c++
class ChessBoard {
  ...
  cout << "your move" << endl;
}
```

This is bad design makes it difficult to reuse / modify the code

what if you want to use the Chessboard - but not communicate using stdin / stdout?



One Sol'n

give the class stream objects that we want it to use

eg:

```c++
class ChessBoard {
  istream &in;
  ostream &out;
public:
  ChessBoard(istream &in, ostream &out): in{in}, out{out} {
    ...
    cout << "your move" << endl; // slightly better = not assuming cout but still assuming in/out streams
  }
}
```



What if we dont want streams at all?

out 

- graphical display
- screen readers 

in

- joystick
- voice

So, our chessboard should not do in/output - arguably, our chessboard should not be doing any communication - b/c its a *chess* board

### Single Responsibility Principle

- a class should only have one reason to change (ie. it should manage one thing)
- fame state / logic + communication are separate

**Better Sol'n** 

- chessboard should manage state and communicate with some external entity, which handles input/output (ie. function calls, parameters, exceptions)

Question:

- should main handle communication? NO
- you want an external entity that is maintainable and reusable - both things you dont get from main

Best sol'n:

- define an external class to manage communication



# Pattern: Module - View - Controller

Separate the data / state of your app from the presentation of that data, and control of data (ie interaction)

- 3 classes / components
  1. Model - data / state of your app
  2. View - presentation / interface
  3. Controller - how interface / data is manipulated



The **Model**:

- can have multiple views (eg: text, graphical) loosely coupled
- does not need to know details
- just needs to notify them when data changes
- often resembles the Observer Pattern - ie. model - subject; view - observer



The **View**:

- just displays, state from the model
- handles notifications, etc as observer



The **Controller**

- mediate between model and view
- may communicate with user for input
- may enforce turn-taking / ordering

Role of Controller - may change based on requirements

- always mediates between modal/view



**Why MVC** ?

provides opportunities for customization and reuse.



# Exception Safety

Consider:

```c++
void f() {
  MyClass *p = new MyClass();
  MyClass mc;
  ...
  g();
  delete p;
}
```

No leaks in the code, as long as it runs normally.

but what if g() raises an exception?



What's guaranteed ?

- during stack (after an exception), all stack-allocated data is cleaned up - destructors run, memory freed
- but, heap-allocated memory is not freed, of g() throws an exception - m is cleaned up, p leaks memory

Fix?

```c++
MyClass *p = new MyClass;
MyClass mc;
try {
  ...
  g();
} catch (...) {
  delete p; // make sure to delete p!
  throw;
}
delete p; // nessessary duplication here. if no exception threw, still need delete p
```

meh, tedious, repetitive; error-prome



In some other languages (not c++), may have a "finally" block when you can specify code that always runs 

eg:

```java
try {
  g();
} catch (...) {
  cout << "ouch!" << endl;
} finally {
  delete p; // always runs! yah! not duplication
} // good feature about java, not c++
```



What does C++ guarantee? (when an exception is raised)

ALL that c++ guarantees is that destructors for stack-allocated objects will run.

so, let's use that

use stack-allocated objects as much as possible...

***C++ Idiom RAII*** - Resource Acquisition Is Initialization

- Every resource should be wrapped by a stack-allocated object and the destructor should just delete /free it

eg:

```c++
// files 
{
  ifstream f{"text.txt"}
  ...
} // out-of-scope? ifstream closes
```

Acquiring the file resource happens during initialization

The file resource happes during initialization

The file is guaranteed to be closed when f is popped from the stack and its destructor runs.



This can be done with dynamic memory as well.

`class std::unique-ptr<T> `

- is a template class that can hold a pointer to an object that you specify when its constructed
- `#include <memory>`
- the destructor will free the pointer
- while in-scope, you can dereference it and use as a pointer

eg:

```c++
void f() {
  auto p = std::make-unique<MyClass>();
  // allocates space for a Myclass object returns a unique-ptr<MyClass>
  MyClass mc;
  g();
}
```

Issue?

``` c++
class C{
  auto p = std::make-unique<C>(); // p points to an instance
  unique-ptr<C> q=p; // error
};
```

what happens when a unique-ptr is copied?

2 objects pointing to same memory - we cannot delete memory twice!

copying is *disabled* for unique-ptrs for this reason - but you can move them

Sample Impl for unique-ptr (in git repo)

```c++
template <typename T> class unique-ptr {
  T *ptr;
public:
  unique-ptr(T *p): ptr{p} {}
  ~unique-ptr() {delete p;}
  unique-ptr(const unique-ptr<T> &other) = delete; // compiler dont create default
  unique-ptr<T> &operator=(const unique-ptr<T> &other) = delete;
  unique-ptr(unique-ptr<T> &&other): ptr{other.ptr} {other.ptr=nullptr} {}
  unique-ptr<T> &operator=(unique.ptr<T> &&other) {
    //swap ...
    using std::swap;
    swap(other.ptr, this->ptr);
    return *this;
  }
  T& operator *() {
	return *ptr;
  }
}
```

