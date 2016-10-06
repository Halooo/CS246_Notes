```c++
struct Node {
  int data;
  Node *next;
};
Node *np = new Node;
// *np is in stack and Node is in heap
delete np;
```



- stack-allocated variables are auto deallocated when they go out of scope (eg: function returns)
- heap-allocated memory is ***not*** auto deallocated - must ~~delete~~ it !

stack allocation

*expensive* code

```c++
Node getMeANode() {
  Node n; // stack-local
  return n; // copies Node n back to callers stacke-frame
}
int main() {
  Node q = getMeANode(); // gets a copy from function, maybe "expensive" to copy
}
```



return a pointer

BAD code

```c++
Node *getMeANode() {
  Node n;
  return &n; // Node descarded on return, invalid address - BAD
}
int main() {
  Node *q = getMeANode(); // will get random stuff
}
```



GOOD code

```c++
Node *getMeANode() {
  Node *np = new Node; // create on heap, dynamic - GOOD
  return np; 
}
int main() {
  Node *q = getMeANode();
  ...
  delete q;
}
```



```c++
Node *getMeANode() {
  Node *np = new Node; 
  return *np; // you can return a reference from a function - but becareful!
}
int main() {
  Node q = getMeANode(); // ?
  Node &q = getMeANode(); // no idea that we need to delete it or not
  ...
  delete q;
}
```



## Operator Overloading 

- defining operators for types that we create 

eg: adding 2 nodes add 2 vectors 

```c++
struct Vec {
  int x,y;
};
```

want to be able to add(+) and multiply(*) by a scalar

```c++
// Vec v1{1,2};
// Vec v2{3,4};
// Vec v3 = v1+v2; // v3{4,6}
Vec operator+ (const Vec &v1, const Vec &v2) {
  Vec v{v1.x+v2.x,v1.y+v2.y};
  return v;
}

// Vec v4 = 2*v3; // v4{8,12}
Vec operator* (const int k, const Vec &v) {
  return {k*v.x,k*v.y}; // OK - compliers knows it is a Vec
  					  // since the return type is indicated above
}

v4 = 2*v3; // OK
v5 = v3*2; // Not OK
Vec operator* (const Vec &v, const int k) {
  return k*v;
}
v6 = v4*2; // OK
```



### Overloading `<<` and `>>`

```c++
struct Grade {
  int theGrade;
};
// cout - eg cout << grade;
ostream & operator << (ostream &out, const Grade &g) {
  out << g.theGrade << "%";
  return out;
}

// cin - eg: cin >> grade >> x >>

istream & operator >> (istream &in, Grade &g) {
  in >> g.theGrade;
  if (g.theGrade < 0) g.theGrade = 10;
  if (g.theGrade > 100) g.theGrade = 100;
  return in;
}
```



## Preprocessor 

- Transform code before the compiler sees it



- `#...` - preprocessor directive - eg. `#include <iostream>` 
- including C headers: new naming conversion 



- `# define VARIABLE VALUE`
  - sets a preprocessor variable 
  - not type-safe
  - preprocessor just does search/replace
- eg:`#define MAX 10` 



- `# define VARIABLE` - no value, but "exists" as a "flag"

how do we use this?

- condition compilation - include / exclude source code based on a flag



```C++
# define BBOS 1
# define IOS 2
# define OS IOS
# if OS == BBOS
long long int key; // discards code block under IOS, keep that under BBOS
# elif OS == IOS 
short int key; // vice versa
# elif ...
cout << key;
# if 0 ...
# endif
```



```c++
int main() {
  cout << X << end; // preprocessor for x?
}
```

`g++ -DX=15 defines.cc -o defines` 

- `-D` directive
- `X=15` define X=15 



eg: including / excluding debugging comments

```c++
int main() {
#if def DEBUG
  cout << "settag x=1" << endl;
# endif
  int x = 1;
  while (x<10) {
    ++x;
    #if def DEBUG
    	cout << "x=" << x << endl;
    #endif
  }
}
```

`g++ debug.cc -o debug`

`g++ -DDEBUG debug.cc -o debug` 



## Separate Compilation 

- split program into compostable modules module having 
  - interface - type definitions, function prototypes (header files - .h files)
  - implementation - full implementation of code (source - .cc files)
- recall: 
  - declaration - asserts existence of something 
  - definition - fill details, allocates memory 

```c++
// vec.h (header)
struct Vec {
  int x,y;
};

Vec operator+(const Vec &v1, const Vec &v2);
Vec operator*(const int k, const Vec &v);
Vec operator*(const Vec &v, const int k);
```

```c++
// main.cc
# include <vec.h>
int main() {
  Vec v1 {1,2};
  Vec v2 {2,3};
  Vec v3 = v1+v2;
}
```

```c++
// vec.cc 
# include <vec.h>
Vec operator+(const Vec &v1, const Vec &v2) {
  ...
}
...
```

