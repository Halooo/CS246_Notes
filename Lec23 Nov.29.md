# How virtual methods work

```c++
class Vec {
  int x, y;
public:
  int doSomething() {...};
}

class Vec2 {
  int x,y;
public:
  virtual int doSomething;
}
```

what the differences (in memory)

```c++
Vec v{1,2};
Vec w{1,2};
// do these look the same in memry
cout << sizeof(v) << " " << sizof(w);
// 8 16
```

2 ints (4 bytes each)

2 ints (4 bytes each) and one vptr (vptr to the vtable)



Virtual Methods

```c++
Book *bp = new comic; // a Text or Book
auto pb = make_shared<Comic>();
pb->isItHeavy(); // virtual method
```

Because IsItHeavy is virtual, compiler figures out at runtime which version of isItHeavy o run

How? 

For a class with virtual functions, the compiler creates a table of function pointers called **vtable** 

- instances of this class also read a ptr to the vtable

```c++
class C {
  int x,y;
  virtual void f();
  virtual void g();
  void h();
  virtual ~C();
}
C c,d;
```



Steps in calling a virtual method

1. from an object, follow the vptr to the vtable
2. fetch the pointer to the actual method from the vtable
3. follow the function ptr and call the function

This happens at *runtime*

so, virtual function calls have a (small) performance cost, relative to non-virtual functions

Also, declaring one or more virtual functions adds a vptr to the object and the associated vtable

- so, classes with no virtual functions are smaller then equivalents 

So, how is an object laid out?

This is compiler dependent 

In g++:

| vptr   |
| ------ |
| field1 |
| field2 |
| ...    |

vptr at top, so compiler doesn't need to know what class this is can always find vptr/vtable

```c++
class A {
  int a,c;
  virtual void f();
};
class B:public A {
  int b,d;
};
```

| vptr |
| ---- |
| a    |
| c    |

| vptr (a* b*) |
| ------------ |
| a            |
| c            |
| b            |
| d            |



# Multiple Inheritance

```c++
class A {
  int a;
}
class B {
  int b;
}
class C: public A, public B {
  void f() {
    cout << a << b;
  }
}
```

A <---|

  	  |---C

B <---|



Challenge:

suppose B inherits from A, C inherits from A

```c++
class D: public B, public C {
  public:
  int d;
};

D dObj;
dObj.a = 5; // which a? the a from B or from C
```

The access to field a is ambiguous 

- is it the a that comes from B or C?
- compiler rejects this code due to this ambiguity 

We need to specify which one:

```c++
dObj.B::a;
dObj.C::a; // but is this what we want?
```

if B and C both want from A, there probably should be one A 

- the default is to get 2 copies - which is wrong

  ​A

  /	\

  B	C

  \	/

  ​D

**deadly diamond of death** (and doom and destruction)



So if we want one copy of A

- avoid ambiguity from base/super class



Make A a virtual base class

use virtual inheritance

```c++
class B: virtual public A {...};
class C: virtual public A {...};
class D: public B, public C {...};
```

​	A <--- one instance of A

/virtual	\virtual

B		C

\		/

​	D



eg: I/O stream hierarchy

img 

```
		  	  ios_base
			 	 |
				ios
		  /(virtual)\(virtual)
	istream			   ostream
   /      |       \	/      |       \
istring ifstream iostream  ofstream  ostring
stream							   stream
				/   \
			fstream	stringstream
```



Re-consider memory

How do we layout memory for multiple inheritance?

vptr

A fields

B fields

C fields

D fields

- just like we did with single inheritance
- goal: pointer to the top
  - `A*, B*, C*, D* ` are all valid for a D object

A - A fields

B - A, B

C - A C

D - A B C D

What does g++ do?

Try something else... different offsets for different pointers



vptr  -  B* - what about A? wrong

B fields

vptr - C* - what about A? wrong, D* - what about A, B? wrong

C fields

D fields

vptr - A* good

A fields



B- for instance - needs to be laid out so that it can find A,B - even if it points at B, a B* cannot find A fields

- issue- no idea where A fields exist reachable to itself. 

eg:

```c++
B b; // B does not know that c and d exit - just knows A and B
```

vptr

B fields

C/D

A fields

so the distance between B fields and A fields isnt known at compile time

- so, B* cannot find A fields

how can B find A fields



The challenge is that a class only knows about its own fields (and base class fields)

- so B should only be aware of B and A
- but "distance" between B and A fields depends on other classes
  - ie. B* cannot figure it out
- Goal: a pointer of a particular type to find it's fields and base class fields
- sol'n:
  - derived class will have a pointer to its base class fields in its v-table(assumes that a B* points to B, D* points to D, etc)
  - pointer assignment between A,B,C,D changes the address of the pointer (ie. for an object of type D, assign to `A*, B* or C*` will change the pointer to the appropriate offset)