# Exceptions (cont'd)

Consider:



``` c++
void f() {
  ...
  throw out_of_range("f"); // raise an exception with a parameter
}

void g() {f();}
void h() {g();}

int main() {
  try {
    h();
  } catch (out_of_range) {
    ...
  }
}
```

What happens?

main calls h

h calls g

g calls f

f `throw` ... all the way back to main



Control back through the call chain (unwinds the stack) until a handler is found (for the exception)

- If there is no matching handler, your program terminates 

What is out_of_range? a class

The statement out_of_range("f"} - invokes ctor, with "f" as a parameter (auxil information)



How to examine auxiliary info?

``` c++
try {
  ...
} catch (out_of_range ex) {
  cout << ex.what() << endl;
}
```



A handler can do part of the recovery job. ie.take corrective action and throw another exception.

``` c++
try {
  ...
} catch (someErrorType s) {
  ...
  throw SomeOtherError (...); // new 
}
```

or just throw some exception

``` c++ 
try {
  ...
} catch (someErrorTpe s) {...
  throw;
}
```



### Why do we say throw instead of throw s>

Try exception s maybe a subclsss of SomeErroType , rather than SomeErrorType intself

- throw s - throws a new exception of SomeErrorType 
  - ie. slices the derived class into SomeErrorType
- throw - rethrows the actual exception (of actual type) - no slicing



When throwing/raising an exception, you should try and be specific

But, you can write a handler to capture all



``` c++
try {.....}
catch (...) { // literaly ... - three dots is generic exception handler
  // do something
}
```

``` c++
try {
  // do sth
} catch (exA) {
  //
} catch (exB) {
  //
} catch (...) {
  // default
}
```



You can throw "anything" - doesn't even have to be an object

eg: git: lectures/c++/exceptions - fib - fact (both recursive)

- using exceptions for regular data passing is a bad idea - overhead
- save exceptions for accasional use /exceptional circumstances (like errors)

You can also define your own exception classes

``` c++
class BadInput {};
try {
  int n;
  if (!(cin>>n)) throw BadInput{};
}
catch (BadInput &) { // return exception by reference
  cerr << "badly formed input" << endl; // avoids slicing problem
  									// compiler knews actual type
  									// maxim: "Throw by value, catch by reference"
}
```



Rule: ***Never, ever*** for any reason, ever, 

- never throw an exception from a destructor
  - dtor may be called after an exception is thrown (while going up the call stack)
  - if your dtor throws an exception, then you have 2 unhandled exceptions - program will terminate 



# Design Patterns (cont'd) 

Best - practices for solving common problems

**Guiding Principle**

- program to interfaces, not implementation
- *Abstract base classes* (virtual methods)
  - define the interface
  - work with pointers to abstract base classes, call methods through pointers
  - concrete subclasses can be swapped in /out
- gives us one common abstraction over a variety of behaviours



eg: Iterators

``` c++
class AbatractIterator {
public: 
  virtual int &operator *() const = 0;
  virtual AbstractIterator &operator ++() = 0;
  virtual bool operator== (const AbstractIterator &other) const = 0;
  virtual bool operator!= (const AbstractIterator &other) const {return !(*this == other);}
  virtual ~AbstractIterator();
}

class List {
  struct Node;
  ...
public:
  class Iterator: public AbstractIterator {
    ... // override /implement all virtual methods
  };
};

class Set {
  ...
public:
  class Iterator:public AbstractIterator {
    ...
  };
};
```



Now, I can write functions that work on any data structure with an iterator derived from AbstractIterator

eg: function to apply a Function from arbitrary iterator /data structure

```c++
template <typename Fn>
  void foreach(AbstractIterator start, AbstractIterator end, Fn f) {
    while (start != end) {
      f(*start);
      ++start;
    }
  }
```

this function works over Lists, sets and anything else with an Iterator derived from `AbstractIterator` 



## Observer Pattern

- publish - subscribe (pub/sub)
- model-view-controller

2 entities:

- one publisher
  -  generates data
- one or more observers
  - monitor data 
  - need to know when the data changes 

eg: spreadsheet - cells with data and graphs (cells publisher, graphs observers)

characteristics

- publisher works alone
- add/remove observers
- loosely coupled
- publisher knows very little about observers

**UML**

![mvc](./img/MVC.png)

Sequence

0. Concrete Observer calls subject::attach to register itself; Subject maintains a list of observer * (eg. Vector \<observer*> )
1. Subjects state gets updated (eg. value in a cell changes)
2. Subject notify Observers() calls each observers notify
3. each observer calls concrete subject::getState to get state and react accordingly 



eg: horse racing & betting

subject - publishes the winner

observer - people betting - observe race, declare victory when their horse wins



``` c++
class Subject {
  vector <observer*> observers;
public:
  subject();
  void attach (observer *ob) {
    observers emplace_back(ob);
  }
  void detach(observer *ob);
  void notifyObservers() {
    for (auto &ob: observers) ob->notify();
  }
  virtual ~Subject() = 0; // need to be implemented Subject::~Subject() {}
};

class Observer {
public: 
  virtual void notify()=0;
  virtual ~Observer() {}
}
```

``` c++
class HorseRce: public Subject {
  ifStream in; // source of data
  string lastWinner; 
public:
  HorseRace(string source): in{source} {}
  ~HorseRace() {}
  bool runRace() {return in >> lastwinner;}
  string getState() const {return lastWinner;}
}

class Better: public Observer {
  HorseRace* subject;
  string name, myhorse;
public:
  Better(HorseRace *hr, string name, string myhorse): subject{hr}, name{name}, myhorse{myhorse}
  {  
    subject->attach(this);
  }
  void notify() {
    string winner = subject->getState();
    cout << (winner == myhourse ? "I win! :D" : "I lost :(") << endl<<
  }
};
```

```c++
// main
int main () {
  HorseRace hr {"source.txt"};
  Better Larry {&hr, "Larry", "Runs Like A Cow"};
  
  while (hr.runRace()) {
    hr.notifyObserver()
  }
}
```



**Simplifications**

1. If the state is trivial, you can publish directly in your notify() and eliminate getState()
2. If you *know* that you will only have one single subject, you can merge subject and concrete subject(horserace class)
   - risky, hard to get generalize later
3. If the subject & observer are the same, could merge into one class. eg: cell is .xlsx (in excel)

