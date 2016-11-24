# Shared Pointer

If you want to be able to copy a pointer, use `std::shared-ptr`



``` c++
void f() {
  auto p1 = std::make-shared <MyClass>(); // allocates space for MyClass object
  if(...) {
    auto p2 = p1;
  } // p2 is out of scope, object is not deleted since p1 still points to it
} // call stack unwinds, p1 out of scope, object is deleted
```



shared-ptrs maintain a reference-count - a count of all shared-ptrs pointing to that object

- memory is freed when reference-count == 0 (ie. number of shared-ptrs is zero)

Use unique and shared pointers - greatly reduces memory leaks.

For other resources, do something similar

- eg. fstream for files, XWindow for windows



# Exception Safety

We have 3 levels of exception safety with f 

``` c++
void f() {
  MyClass *p = new MyClass;
  MyClass mc;
  g(); // throws?
  delete p;
}
```

1. **Basic Guarantee** - if an exception occurs, the program will be left in a valid but unspecified state; nothing is leaked, class invariants are maintained.
2. **Strong Guarantee** - if an exception is raised while executing f, the state of the program is left as if f was never called.
3. **No Throw** - f will *never* throw an exception, and will always accomplish its task.



``` c++
class A {...};
class B {...};
class C {
  A a;
  B b;
public:
  void f() {
    a.method1(); // may throw, strong guarantee
    b.method2(); // may throw, strong
  }
};
```

is `C::f()` exception safe?

1. if a.method1() throws an exception
   - first statement nothing has happened yet
   - call stack unwinds, nothing happens
   - ok ~ strong guarantee
2. if `b.method2()` throws an exception
   - `a.method1()` has run already
   - to enforce a strong guarantee, I need to *undo* what `a.method1()` did
   - *very* hard to do - esp. if method 1 has "non-local side effects"

so, f() is probably not exception safe (b/c I cannot undo from example)

But, if `A::method1` and `B::method2` do ***not*** have non-local side effects, we *can* provide exception safety with **copy-swap** 



```c++
class C {
  A a;
  B b;
public:
  void f() {
    A atemp = a;
    B btemp = b;
    atemp.method1(); // assumption - no non-local side-effects
    btemp.method2(); // ie. a and b are self contained
    a = atemp;
    b = btemp;
  }
}
```

Because copy-assign can still (potentially) throw an exception, we dont yet have exception-safety

- would be better if the swap was guaranteed to not throw an exception (ie. no-throw guarantee)
- **a non-throwing swap mechanism is the key to writing exception safe code** (will be on exam)



**key observation** - copying pointers cannot throw (ie. always works)

so, how do we build a swap that uses pointers?

eg: pImpl idiom

``` c++
struct CImpl {
  A a;
  B b;
};
class C {
  unique-ptr<CImpl> pImpl = make-unique<CImpl>();
public:
  void f() {
    auto temp = make-unique<CImple>(*pImpl);
    temp->a.method1();
    temp->b.method2();
    swap(temp, pImpl); // no-throw
  }
}
```



if A::method1 or B::method2 offer *no* exception safety, the f() cannot offer either



### Exception Safety & STL vectors

Vectors

- encapsulate a heap-allocated array
- follow RAII - when a static-allocated vector goes out-of-scope, the internal heap allocated memory is freed

eg. 

```c++
void f() {
  vector <MyClass> v;
  ...
} // v out of scope, MyClass dtor runs on all objects in v, finally, v's memory is freed
```



but,

```c++
void g() {
  vector <MyClass*> v;
  ...
} // out-of-scope, pointers do not have destructors to run, only v's memory is freed
```

In this case, objects pointed to by pointers in v are not freed

- vectors does not understand the ownership of memory by pointers (ie. if it can free memory)

so, if these objects are to be freed, you have to do it!

``` c++
void g() {
  vector <MyClass *> v;
  ...
  for (auto &x:v) delete x; // old, manual, pedantic way
}
```



fancy way! using shared pointers

```c++
void h() {
  vector <shared-pointer<MyClass>> v;
  ...
} // v goes out-of-scope - shared-pointer destructor runs, objects freed if no other shared-ptrs
```

good, no explicit deallocation 



Consider:

`vector<T>::emplace_back`

- offers strong guarantee
- if array is full (ie. cap == size)
  - allocate a new larger array
  - copy the objects over (copy ctor)
    - if a copy ctor throws exception
      - destroy the new array
      - leaves the old array intact
    - else
      - delete the old array

**But**, copying is expensive, the old data is getting thrown away

- moving from old to new is more efficient
  - allocate a new larger array
  - move objects from old to new (move ctor)
  - delete the old array

**But** more problems, the move ctor can throw exception as well, cannot offer strong guarantee for `emplace_back`

- Safety of `emplace_back` relies on safety of move ctor move ctor-needs to be no-throw.
- this would mean that we no longer have strong guarantee for emplace_back

**If** the move ctor offers no-throw (it wont throw on exception), emplace_back will use move

- otherwise it will use copy

your move operators should provide no-throw guarantee (if possible) 

- we can do this (with pImpl, move/swap pointers will not throw an exception)
- need to tell compiler that they wont throw with `noexcept`

eg:

```c++
class MyClass {
public:
  MyClass(MyClass &&other) noexcept {...}
  MyClass & oporator=(Myclass &&other) noexcept {...}
}
```

for us, this is a means to ensure that emplace_back can use move operators - and still provide the strong guarantee

- your move operators should off no-throw
- if you know a function will rather throw an exception, declare it noexcept
  - at a min, move/swap are always be noexcept
  - facilitates optimization for the compiler



