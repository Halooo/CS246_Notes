last class: default function parameters

see git, `lectures/c++/functions/default`

```c++
void printSuit(string name = "suit.txt"){
  ...
}
```





## Function Overloading

in C:

``` C
int negInt(int n) {return -n;}
int negFloat(float f) {return -f;}
int regBool(bool b) {return !b;}
```

in C++:

function can have the same name, differ only by parameter lists

```c++
int neg(int n) {return -n;}
int neg(float f) {return -f;}
int reg(bool b) {return !b;}
```

- compiler uses the umber and type of parameters to figure out which function to call
  - must differ in arguments not just return type

eg:

```c++
bool neg(int a);
int neg(int a); // will not work (very difficult for compiler)
```

```c++
cout << x
```

`cout` - i/o stream

`<<` - function

`x` - int/float/string



## Constants

```c++
const int x = 5; // cannot reassign, must be initialized
const int y; // BAD :(
```



## Structures (Struct)

```c++
struct Node { // a node in linked list 
  int data;
  Node * next; // in C, struct Node - struct keyword not needed here in C++
}; // ";" at the end :D
```

initializing structs:

```c++
Node n1 {5,nullptr}; // assigns values to members of struct in-order
```

`nullptr` - new in c++, replaces NULL for pointers (c)

`Node n2 = n3; // copy data in n3 to n2 `

`const Node n4 = n1; // n4 cannot be changed`  



## Parameter passing

recall:

```c++
void inc(int n) {n++;}
int x = 5;
inc(x);
cout << x << endl; // 5
// pass-by-value
```

pass-by-value 

call-by-value

- `inc()` gets a copy of the parameter x, and increments the copy
  - original is not changed

pass-a-pointer

- if you want to modify the original

eg:

```c++
void inc (int *n) {
  *n++; // original data is modified
}
int x = 5;
inc(&x); // x's address is passed by value
cout << x << endl; // 6
// pass-a-pointer
```

c++ has another pointer-like type:

## Reference 

- sort of like an alias to a variable

```c++
int y = 10;
int &z = y; // z is an l-value reference to an int y
		   // like int *const z = &y;

cout << y; // same IO
cout << z; // same IO
z = 5;
cout << y; // 5
```

l and r-values: 

`int x = 5`

`int x` l-value, have an address

`5` r-value, may or may not have an address



Things you cannot do with l-value references (&)

- cannot leave them uninitialized (ie, `const`) 
- must initialize with something that has an address (l-value), basically refs are pointers.

eg: 

```c++
int &x = 3; // BAD :( - NO ADDRESS
int &x = y + z; // BAD :( - NO ADDRESS
int &x = y; // OK :)
```



- cannot create a pointer to a reference (ie. ref doesn't have own address)

eg:

```c++
int &*x; // BAD ptr to ref
int *&x; // OK  ref to ptr (ie. alias to a pointer)
```

- cannot create a reference to a reference
  - `int && x; // BAD` -- `&&` meaningful but not here
- cannot create an array of references 
  - `int &r[3]= {a,b,c};` -- use pointers

What are they good for?

function pointers

```c++
void inc (int &n) { // pass by reference
  n = n + 1; // no de-referening required
}
int x = 5;
inc(x); // no address pointer
cout << x; // 6
```

eg: overloading `cin` and `cout`

```c++
cin >> x; // takes parameters as references
std::istream & operator >> (std::istream & in; int &n);
```

`std::istream &` return cin

`operator >>` function 

`std::istream & in;` cin

`int &n` x

overloaded for different types: `com >> int` `com >> float` `com >> String` ...



## Cost / Performance

pass-by-value (eg:` void f(int n)`) copies its arguments

- so if the argument is big this can be costly - memory and time to allocate

eg:

```c++
int f(int n) {...} // cheap!
struct RreallyBig {...}; // 1GB
int g(ReallyBig bg) {...} // huge cost
```

pass-by-reference 

- avoids these costs
- arg so does pass-by-ptr

eg:

```c++
int g(ReallyBig &bg) {...} // faster
						 // risk - data is exposed reference to original
```

eg: 

```c++
int h(const ReallyBig &bg) {...} 
```

advice: 

prefer pass-by-const ref when possible 

- all read-only parameters should be this

exceptions?

- passing a single int/bool
- want a copy - pass-by-value



***Strange Behavior of Reference***

```c++
int f(int &n) {...}
int g(const int &n) {...}
f(5); // NO! not an l-value
g(5); // OK??? creates a temp space in memory
```



## Dynamic Memory

in C:

```c
int *p = malloc(1*sizeof(int)); // # of bytes
free(p);
```



in C++:

- use new, delete operators instead
- type-aware -- less error-prone

```c++
struct Node {
  int data;
  Node *next;
};
Node *np = new Node; // allocates space for Node
delete np;
```



```c++
int x;

Node *np = new Node;
```



- all local variables reside on stack 
- variables are de-allocated when they go out-of-scope (eg: function returns)
- allocated memory resides on the heap
- remains allocated until delete us called 
  - if do not de-allocate? memory leak - BAD :( - exhaust memory, fragmented, crash.



## Array form

```c++
Node *myNodes = new Node[10];
delete [] myNodes; // if no [], will delete one Node and result in memory leak
```

- do not mix up with new/delete



```c++
Node getMeANode() { // creates uninitialized node
  Node n; 
  return n; // returns on stack - maybe expensive
}

// return a pointer/ref
Node* getMeANode() { // returns invalid pointer
  Node n;
  return &n;
}
```

