# Visitor Pattern

eg: game with turtles, bullets 

- used for implementing double-dispatch
- virtual methods are chosen based on the run-time type of the object
  - but what if you want to choose based on **2** objects?
  - ie pick a virtual method to run based on **2 ** types -double-dispatch

We want something like this:

`virtual void strike(Enemy &e, Weapon &w);`

- of we attach to an enemy, then its tied to the type of enemy

  (ie. choosing based on type of enemy)

  `virtual void Enemy::strike(Weapon &w);`

- similarly, we cannot just attach to the weapon, either

  ie `virtual void Weapon:: strike(Enemy &e);`



The trick to dispatching based on **both** entities (double dispatch) is to combine *overriding* and *overloading*

eg:

```c++
class Enemy {
public:
  virtual void beStruckBy(Weapon &w)=0;
};
class Turtle:public Enemy {
public:
  void beStruckBy(Weapon &w) override {w.strike(*this);}
};
class Bullet:public Enemy {
public:
  void beStruckBy(weapin &w) override {w.strike(*this);}
};
```

```c++
class Weapon {
 public:
  virtual void strike (Turtle &t)=0;
  virtual void strike (Bullet &b)=0;
};

class Strike {
 public:
  void strike (Turtle &t) {...} // strike + stick
  void strike (Bullet &b) {...} // bullet + stick
};
// similar for Rock
```

- usage

```c++
Enemy *e = new Bullet();
Weapon *w = new Rock();
e->beStruckedBy(*w); // Bullet struck by Rock
```

ie

- `Bullet::beStruckBy` runs (virtual)
- it calls `Weapon::Strike`, where *this is a Bullet
- Bullet version of strike is called
- virtual method Strike(Bullet &b) resolves to Rock::strike(Bullet &b)



Visitor can be used to add functionality to existing classes. without changing or recompiling the class

eg: add a visitor to the Book hierarchy 

AbstractBook

|___Book

|___Text

|___Comic



BookVisitor

```c++
class AbstractBook {
 public:
  virtual void accept (BookVisitor &v)=0;
};
class Book: public AbstractBook {
 public:
  vvoid accept (BookVosotpr &v) override {v.visit(*this);} // beStruckBy
};
class Text: public AbstractBook {
 public:
  void accept(BookVisier &v) override {v.visit(*this);}
};
class Comic: public AbstractBook {
 public:
  void accept(BookVisier &v) override {v.visit(*this);}
}
```

```c++
class BookVisitor {
  virtual void visit (Book &b)=0; // overloaded
  virtual void visit (Text &b)=0;
  virtual void visit (Comic &b)=0;
}
```

What is this good for?

eg: I want to count my types of books 

- count Books by author
- count Text by topic
- count Comic by hero

I **could** do this with a `map<string, int>`

If I did this, I'd need to add:

`virtual void updateMap(map<string, int> &m); // add as a static method`

**or** we can model using visitor:

```c++
class Catalog: public BookVisitor {
  map<string, int> thecatalog;
 public:
  void visit(Book &b){++theCatalog[b.getAuthor()]};
  void visit(Text &t){++theCatalog[t.getTopic()]};
  void visit(Comic &c){++theCatalog[c.getHero()]};
}
```

in headers:

book.h(text.h, comic.h)

|_includes BookVisitor.h

​			|_includes text.h

​						|_includes book.h

circular dependency



**But**, we have header guards (who cares 误)

- it works ... by preventing text.h from including book.h
- Text does not get a copy of Book, so compiler does not understand reference to Book

So, we have a bunch of #includes to define our dependencies (in .h files)

Q: are all these includes needed?

# Compiler Dependencies

When does an actual compiler dependency exist? (ie. when do we need an #include)

consider:

```c++
class A {};
class B: public A {};
class C {
  A myA; // true dependency -needs size of A
};
class D {
  A *myA; // not a true dependency - pointers are fixed size - foward decl'n
};
class E {
  A f(A x); // not a true dependency - forward decl'n works. eg: class A; (instead of #include "a.h")
}
```

So,

- true dependency - implies #include is needed
- not dependency - implies a forward declaration is ok

if a compilation dependency isnt required, dont introduce one by using #include - leads to circular dependencies

In our example, if class A changes, only class A B C  need recompilation



We can use forward decl'n for clases D E, but in the implementation the dependency is stronger(ie. #include required in impl'n file)



eg: 

```c++
// d.cc
#include "a.h"
void D::f() {
  myApointer->someMethod(); // need to know more about A, true compiler dependency
}
```

**Rules**:

Do the #include in .cc file, instead of .h file (if possible)

- use forward declaration in header
- .cc files never include each other, so no risk of circular dependencies 

To fix Book example?

- in header files, replace #includes w/ forward declarations



**Consider** the Xwindow class

```c++
class XWindow {
  Display *d;
  Window w;
  int s;
  GC gc;
  unsigned long colors[10];
  // private data - you should not need to know about this - should be hidden
 public:
  ...
};
```

What if we wanted to change a private field?

- everyone needs to recompile - data mixed with implementation
- like to hide details away

**Sol'n**

use the plmpl idiom (pointer-to-implementation)

Create a second class XWindowlmpl

```c++
// XWindowPlmpl.h
#include <X11/Xlib.h>
struct XWindowImpl {
  Display *d;
  Window w;
  int s;
  GC gc;
  unsigned long colors[10];
}
```

```c++
// window.h
class XWindowImpl; // forward decl'n to the impl.class
class XWindow {
  XWindowImpl * pImpl; // no compiler dependency on XWindowImpl.h
 public:
  
};
```

``` c++
// window.cc
#include "window.h"
#include "XWindowImpl.h"
XWindow::XWindow(...): pImpl {new XWindow Impl} ... {}
```

Change how you reference fields:

d `pImpl->d`

w `pImpl->w`

s `pImpl->s`

change in all the methods



- If you confine all private fields within XWindowImpl, the only window.cc needs to be recompiled if you change XWindows Impl




**Generalization** - you can "easily" support multiple window implementations



![windowimpl](./img/windowimpl.png)



The plmpl ido=iom with a class hirarchy of implementations is called the **Bridge Pattern**



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

Goal - Low Coupleing