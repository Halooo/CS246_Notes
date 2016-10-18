# CS246 Lec. 1

### Graphical shell

- launch programs
- unteract with files/folders
- user interface

### Command shell

- text
- launch programs
- etc.

command prompt - win

terminal -mac

bash - Linux

### Commands

Examples

- pwd - print work dir
- cd - change dir
- ls - list
- wc - 
- man - manual

Control signals

- Ctr+D EOF
- Ctr+C break

Input redirection 

- cat filename - cat opens, reads, closes file
- Cat - waits for input from shell (bash)
- cat < input.txt -shell redirects input to the command
- head number - opens and read first number lines of file

Output redirection

- \> redirect output
- it creates or overwrite a file
- \>> append 

RegEx

- \* wild card
- ? any single char
- [] terms

Redirection

- stdin(<) -> program -> stdout(>)/stderr(2>)

## Pipes

taking output from one program using as input for another

use `|` to pipe, eg: 

```
sort words | uniq | wc -w
```

# Pipe

use output from one command as input for another command 

`cmd1 | cmd2 | cmd3 `

eg: 

`cat myfile | wc -w`

`cat words | sort | unique`

- can also use output from one command as parameter

`echo "abc"`



# RegEx

### Pattern matching

`*` - filename match any number of chars

- eg: ls *.txt

`?` - match single character 

- eg: ls sample?.txt

**Regular Expression**

- pattern for matching a string
- eg: web validation in JavaScript, text editor

### egrep (grep)

`egrep <condition> <file>` process the file one line at a time, print out lines matches the condition

eg: `egrep "cs246" index.html`, it will find lines contain string "cs246"

### regex rules

`()` to create a sub-expression

- eg: `(cs|CS)246` matches cs246 or CS246

  `(c|C)(s|S)246` matches cs246 or CS246 or cS246 or Cs246

`[]` match any one cha in brackets

- eg: `[cC][sS]246` cS246

  `[a-z][0-9]` can use for ranges

  `[^0-9]` single char *not* 0-9

  `[^A]`  *not* A

`.` - single character

`^` - start of file

`$` - end of file

### frequency of characters

`?` - 0 or 1 of preceding pattern. eg: `A?` 

`*` - 0 or more of 

`+` - 1 or more 



**eg:**

`egrep ".*" file.txt` any string

`egrep "^cs246" file.txt` match string "cs246" at start

`egrep "^.+$" file.txt`

`egrep "^(..)*$" file.txt` lines of even length



**sample questions:**

list files in curr dir whose name contains a single "a"

`ls | egrep "^[^a]*a[^s]*$"`

fetch words from global dict that start with "e" and consist of 5 chars

`egrep "^e....$"`



# File System

- files and directories structured in a tree

- top node is called root

- reference file / directories by their position in tree

  - eg: /u4/hello - absolute path specified relative to that

- every file can be specified with absolute path

- "working dir" 

  - directing that shell is using
  - can **point at** any directory
  - cd - other dir 
  - pwd - print curr dir

- **Relative path** - path to file or dir relative to working dir

  - for relative paths - `.` represent current dir and `..` represents parent dir

- | d                           | rwx rwx rwx     | 1          | h79sun | h79sun | 25    | Sept.9   | file.txt |
  | --------------------------- | --------------- | ---------- | ------ | ------ | ----- | -------- | -------- |
  | type: **-** file, **d** dir | permission bits | hard links | owner  | group  | bytes | mod date | name     |

In Unix, every file has 

- owner - primary user for this file (creator) and has all permissions

- group - set of one or more users with specific permissions

- | rwx          | rwx          | rwx   |
  | ------------ | ------------ | ----- |
  | user (owner) | group member | other |

  permission bits

- meanings of permission bits

| bit  | file                          | dir                                      |
| ---- | ----------------------------- | ---------------------------------------- |
| r    | file contents can be read     | dir contents can be read, eg: ls         |
| w    | file contents can be written  | dir contents can be modified, eg: create a new file |
| x    | file can be executed as a pro | dir contents can be navigated, eg: cd in to do "stuff" |

### chmod - set permissions on file or dir

|chmod|u|+|r|
------|
||g|-|w|

# Permissions

| rwx  | rwx  | rwx  |
| ---- | ---- | ---- |
| u    | g    | o    |

- file permissions- apply to the file
- dir permissions - apply to dir **contents**
- what if they conflict?
  - will follow the most restrictive set of permissions

# Shell Scripts

### scripts programming

- File containing a set of **shell commands **, executed as programs

#### first. bash

- `#!/bin/bash`   -- can be used to print date, current user, work dir, etc. "she-bang"
  - have to make sure that the file is executable (`chmod u+x <filename>`)

#### Variables

- `x=1` note: without space chars. command sets x=1
- `echo $x` prints value of x
- `echo ${x}` avoids ambiguity, such as `${abc}` and `${a}bc`
- Variables are strings
- `" "` bash expands variables in double quotes: eg: `${PATH}`expands
- `' '` single quote are used for literal

#### Special variables in a script

- do not apply to command line
  - `$1` - represents first command line arg
  - `$2` - represents second command line arg
  - `$0` - name of script
  - `$?` - status of last executed program

**Examples**

check if a word is in the dictionary

```bash
./isItAWord word
isItAWord.bash
#!/bin/bash
egrep '^$1$' /user/share/dict/words
```



check if a word is a "good" password

```shell
if [ condition ] ; then
	...
elif
	...
else
	...
fi
# require space around square brackets []
```



```shell
#!/bin/bash
egrep '^$1$' /user/share/dict/words > /dev/null # /dev/null through away stdout
if [ $? -eq 0 ] ; then
	echo "Bad password"
else
	echo "Good password"
fi
```



Verify number of arguments

```shell
#!/bin/bash
usage() {
  echo "usage: $0 password"
}
if [ $# -ne 1 ] ; then
	usage
	exit
fi
```



## Variables in bash are strings

- can be manipulated as integers or strings

- choice of operator tells bash how to process

- | as integers                              | as strings |
  | ---------------------------------------- | ---------- |
  | `-eq` equals                | `=` equals |            |
  | `-ne` not equals            | `!=` not equals |            |
  | `-lt` less than             | `<` "less than" |            |
  | `-gt` greater than          | `>` "greater than" |            |
  | `-le` less than equal to                 |            |
  | `-ge` greater than equal to              |            |

## Loops

- eg: print numbers from 1 to $1     `./count 10`

```shell
#!/bin/bash
x=1
while [ $x -le $1 ]; do
	echo $x
	x=$((x+1)) 
done
```

eg: iterating over a list (of strings, separated by spaces)

- rename a group of files from .cpp tp .cc

```shell
#!/bin/bash
for name in *.cpp; do
	mv ${name} ${name%cpp}cc
done
```

eg: how many times a word($1) occurs in a file (\$2)

```shell
#!/bin/bash
x=0
for word in $(cat $2); do
	if [ $word = $1 ]; then
		x=$((x+1))
	fi
done
echo $x
```

```shell
#!/bin/bash
for j in $(ls); do
	echo "item $j"
done
```



eg: Payday is on the last Friday of the month, compute this month's payday

syntax: ./payday



```shell
#!/bin/bash
answer(){
  if [ $1 -eq 31 ]; then
  	echo "This month: the 31st"
  else
  	echo "This month: the ${1}th"
  fi
}
answer $(cal | awk '{print $6}' | egrep [0-9] | tail -1)
```

`./payday` -- current month

`./payday march 2016` -- some other month

------

`./payday` -no cmd line args

​			$1 - date

​			$2 - nothing



**functions cannot access command-line arguments **



# Testing

- essential part of software development
  - ongoing - not just at the end
  - begin *before* you start coding -- understand requirements and behaviours
  - should be building "test suits" -- set of tests that document expected behaviour
- testing is not debugging -- testing needs to be systematic and reproducible
- testing cannot guarantee correctness
  - can only prove "wrongness" -- ie. match expected behaviour
- ideally, you should have a separate tester. ie. dev/tester are different
  - bias exist, want it to be objective



### Human testing vs. machine testing

- Human testing 
  - people look at your code, find flaws
  - include code inspections, walkthroughs
- Machine testing
  - run program with selected input, check against expected output
    - check expected behaviour
    - cannot check everything - choose test cases carefully
  - type: black box/grey box/white box
    - black-box: testers have no knowledge of internals - 
    - grey-box: some limited access to implementation
    - white-box: full access to implementation/complete knowledge
- start with black-box
  - check various classes of input.  eg: numeric, +/-
  - boundaries of valid ranges ("edge cases"). min/max value
  - mult. simultaneous boundaries ("corner cases")
  - extreme cases - outside expectations - give binary file instead of int
- white-box testing to supplement
  - ensure execute all logical paths in a program (complete?) - add more test cases
  - make sure every function runs

# C++

invented in 1979 by Bjarne Stroustrup

- working with simula (00 lang)
- add 00 features of similar to C
- "C with Classes" C++ 1983
- C++14 standards are used. C++17 is on its way

```c
# include <stdio.h>
int main() {
  prinf("Hello world \n");
  return 0;
}
```

```c++
# include <iostream>
using namespace std;
int main() {
  cout << "Hello world" << endl;
  return 0;
}
```



- main must return an `int` in C++
- c++ I/O -uses a header `<iostream>`
- output:
  - `std::cout << ... << ... << std::endl`
- `namespace` is a way of grouping (functionality)
- `using namespace std;` lets us say `cout`/`endl` without say `std::cout` / `std:: endl`
- return statement -returns a status code ($?)
  - if return statement is left out, main will return 0;

### Compile

In the student.cs env.

`g++-5 -std=c++14 program.cc -o program`



### Input / Output

- I/O streams
  - cout - printing to standard output (stdout)
  - cin - reading from standard in (stdin)
  - cerr - printing to standard error (stderr)
- I/O operators 
  - `<<` put to. eg: `cout << x;`
  - `>>` get from. eg: `cin >> y;`
- eg add two numbers

```c++
# include <iostream>
using namespace std;
int main() {
  int x,y;
  cin >> x >> y;
  cout << x+y << endl;
}
```



- `cin` ignores whitespace 
- `cin >> x >> y` get two integer from cin assign to x,y
- what if input does not contain integer? -- error
- what if input terminates early (ctr+D) -- error
- we can check for this 
  - `cin.fail()` -- return true on failure of prev. operation
  - `cin.eof()` -- return true on EOF
  - if EOF is reached, both will be true but EOF is triggered after `fail()`

eg: read all ints from stdin and echo one per line stop on bad input or EOF

```c++
# include <iostream>
using namespace std;
int main() {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail()) break;
    cout << i << endl;
  }
}
```

**Read ints from stdin and echo one per line**

```c++
int main() {
	int i;
  while(true) {
    cin >> i;
    if (cin.fail()) break;
    cout << i << endl;
  }
}
```

if cin fails on bad input 

- fail bit/
  - cin.fail() is true

if cin fails on EOF

- fail bit 
- EOF bit
  - cin.eof() true

**read ints simplified**

```c++
int main() {
  int i;
  while(true) {
    if (!(cin >> i)) break;
    cout << i << endl;
  }
}
```

```c++
int main() {
  int i;
  while(cin >> i) {
    cout << i << endl;
  }
}
```



**read ints and handle bad inputs**



# Stream Abstraction

eg: file I/O using streams

c

```c
# include <stdio.h>
int main() {
  char s[256];
  FILE *file = fopen("suite.txt", "r");
  while(true) {
    fscanf(file, "%255s",s);
    if(feof(file)) break;
    printf("%s\n", s);
  }
}
```



c++ 

```c++
# include <iostream>
# include <fstream>
using namespace std;
int main() {
  ifstream file {"something.txt"}; //open file stream
  string s;
  while (file >> s) {
    cout << s << endl;
  }
}
```



last class: default function parameters

see git, `lectures/c++/functions/default`

```c++
void printSuit(string name = "suit.txt"){
  ...
}
```





## Function Overloading

in C:

```C
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



```c++
#include "stdio.h" // look into local dir
#include <vec.h> // look into system
```



```c++
// vec.h
struct Vec {
  int x,y; // declar
};
```

```c++
// vec.cc
#include <vec.h>
...
```

```c++
// main.cc
#include <vec.h>
...
```

Issue: wont compile

- vec.cc - missing main
- main.cc - missing vec implementation 

Processer -> compiler -> linker

- pass through src code
- compiler: compile each independently 
- linker: puts all together

`g++14 -c main.cc // main.o` 

`g++14 -c vec.cc // vec.o`

`g++14 main.o vec.o -o main // exe` 

vec.h? - do not compile this



## Global Variable

eg. variable 

```c++
// intuitive 
// abc.h
int globalNum; // decl + definition
```

- cannot just include in header - source files each get their own def'n 

```c++
// abc.h 
extern int globalNum; // declaration only (defined elseware)
```

```c++
// main.cc
#include <abc.h>
int main() {
  int globalNum = 5; // definition - one source file only
}
```



eg. linear algebra module using vector class

BAD

```c++
// linalg.h
#include <vec.h>
...
```

```c++
// linalg.cc
#include <linalg.h>
#include <vec.h>
```

```c++
// main.cc
#include <vec.h>
#include <linalg.h>
int main() {
  ...
  Vec v;
  ...
}
```

- this will not compile 
  - main.cc, vec.cc each gets multiple copies of vec.h
  - problem ? struct (Vec) is defined multiple times - BAD
- issue -need to prevent headers from being included multiple times
- sol'n 1 - be careful -- i.e manually track dependencies -- nearly impossible 
- sol'n 2 - header guard / include guard

eg. include guard for vector

```c++
// vec.h
#ifndef _VEC_H_
#define _VEC_H_
struct Vec {
  int x,y;
}
#endif
```

guidelines

- always use an include guard in header files
- **never,ever** put `using namespace std` in a header file

# Classes

- "big innovation" in OOP
- idea that structs can also have functions(behavior) 
  - data + behavior (in a struct)

def'ns 

- class 
  - struct type that can potentially contain functions 
- object 
  - an instance of a class
  - **member variables **- instances of variable for an object 
  - **member functions **- functions inside a class

eg. 

```c++
struct Student {
  int assns, mt, final_grade;
  float grade() {
    return 0.4 * assns + 0.2 * mt + 0.4 * final_grade;
  }
};
```



- grade() is a function that can be called 
- assns, mt, final are member variables that hold values for an instance of student
- `Student billy {70,80,22}; // assns,mt,final`
- `cout << billy.grade(); // uses Billy's data`

Formally:

- methods take an extra "hidden" parameter, called ***this*** which is a pointer to the instance of a class
- can rewrite like this:

```c++
struct Student {
  int ...;
  float grade() {
    return 0.4 * this->assns +
    	0.2 * this->mt +
    	0.4 * this->final_grade;
  }
};
```



Initializing objects 

```c++
Studentt billy {70,80,90};
// numbers are assigned ti data members in order
// 70 -> assns
// 80 -> mt
// 90 -> final
```



define a method for initialization - constructor 

```c++
struct Student {
  int assns, mt, final_grade; // same
  float grade() {...}
  Student (int assns, int mt, int final_grade) {
    this->assns = assns;
    this->mt = mt;
    this->final_grade = final_grade;
  }
};

Student billy {60,40,12}; // calls constructor for int
```

- if a constructor is defined, this calls the constructor
- if a constructor is not defined, direct initialization

### uniform initialization

- different forms of initialization, same effect

```c++
int x = 5;
string s = "hello";

int x {5};
string s {"hello"};
Student jane {95,85,98};
```



advantage of constructors

- default parameters can be applied/used
- overloading 
- sanity checks
- logic

default constructor

- if you dont define any constructors the compiler will create one for you
- default - no argument constructor 
  - eg. Student() i.e no argument passed
- what does default do?
  - default constructs objects 
  - get the effects to direct init.

```c++
sturct Student {
  int assns, mt, fi;
  float grade() {...}
}; // no constructor, just default
Student rena; // struct allocate call default constructor
Student tennie {10, 20, 30}; // default, initializes
```



# Using GDB

GNU debugger

part of GCC suite (g++)



To use it:

1. g++14 -g main.cc -o main
   - -g: flag
2. gdb main
3. do stuff with it
   - list
   - break
   - step
   - set

try to google: GDB cheat sheet for more commands 



### Constructors

- the compiler defines a no argument default constructor
- will ***not*** create one if you define any constructor

initializing consts & refs

```c++
struct Strudent {
  int assns, mt, final;
  const int id;
  ...
  
};
```

dilemma: 

1. need to init constant when declared (can't reassign value)
2. but every instance of `Student` needs a unique id

#### When do we init id?

how does an object get created?

1. spare/memory gets allocated
2. fields are constructed. eg. `int assns = 0`
3. constructor body runs <- too late to init constants 

how do we initialize const in step 2?



### Member Initialization List (MIL)

- allows initialization before ctor (constructor) body

```c++
struct Student {
  int assns, mt, final_grade;
  const int id;
  Student (int id, int assns, int mt, int final_grade): id{id}, asns{assns},
  int{mt}, final_grade{final_grade}
  {
    
  }
  // name {inside} : parameter
  // name outside{} : field
}
```



Guidelines:

1. you can and should initialize any fields this way
2. Fields are initialized using the MIL in the order they appear in the class

eg.

```c++
struct Student {
  int final_grade, mt, assns;
  const int id;
  Student (int id, int assns, int mt, int final_grade)
    : id{id}, assns{assns}, mt{mt}, final_grade{final_grade}
  {
    
  }
}
```

tip: just use MIL for init

- more complex code goes in ctor

1. the MIL can be more efficient then initializing in the body of the ctor

   - for fields that are objects and ***not*** in the MIL, they will get default-constructed(and the npotentially initialized again in the ctor body)

2. ***EMBRACE*** MIL, use it!

3. What happens if a field is initialized both in the class and MIL?

   - MIL wins
   - ie. initialization in MIL takes precedence over field init

   eg.

   ```c++
   struct Vec {
     int y = 0;
     const int x = 0;
     Vec(int x, int y): x{x}, y{y} {
       ...
     } // MIL is applied before const int x = 0;
     // so MIL will take the effect
   };
   ```

Constructor:

```c++
Student billy {55,60,70};
Student bobby = billy;
```

how does initialization happen for bobby?

- copy values from fields in billy to fields in bobby
- **copy constructor** - used when one object is initialized as a copy of another object

Every class cares with :

1. default ctor(default constructs all fields that are objects; not created if you define any ctor)
2. copy ctor (just copies all fields)
3. copy assignment operator
4. destructor
5. move ctor
6. move assignment operator

#### copy ctor - copies values

to build your own:

```c++
struct Student {
  int assns, mt, final_grade;
  Student(int assns, int mt, int final_grade): assns{assns}, mt{int}, final_grade{final_grade} {
    
  }
  
  Student(const Student &other): assns{other.assns}, mt{other.mt}, final_grade{other.final_grade} {
    
  }
};
// this is equiv to the default copy constructor
```



When do the default copy ctor fail? A: **dynamic memory**

```c++
struct Node {
  int data;
  Node *next;
  Node(int data, Node *node): data{data}, next{next} {
    
  }
};

Node * n = new Node{1, new Node{2, new Node{3, nullptr}}}; // LL of 3 nodes
Node m = *n; // copy
Node *p = new Node{*n}; // copy
```



stack         heap

n   | -|>     |1|-|> |2|-|> |3|/|

m |1|-|               >

p |-|>        |1|- |>



`m` copied the first node of n in heap and stored in stack. next is pointing to the 2nd node of `n` in heap

`p` creates a new pointer in stack, pointing to a copy of the first node of  `n` in heap, and next is point to the 2nd node of `n` in heap

- copying values like *this* is a "shallow copy"
- we want to **deep** copy that copes the whole list

```c++
struct Node {
  ...
  Node(const Node &other): data{other.data}, next{other.next ? new Node{*other.next} : nullptr} {
    
  }
}
```

When is a copy ctor called?

1. when an object is initialized by another object
2. when an object is passed by value
3. when an object is returned from a function

\* be careful with ctors that take a single parameter

```c++
struct Node {
  ...
  Node(int data): data{data}, next{nullptr} {
    
  }
};
```

- single -argument ctors can lead to implicit behavior

eg

```c++
Node n(4); // OK - calls ctor
Node n = 4; // Compiler: hmm... - maybe you want a Node? implicit conversion from int -> Node
int f(Node n) {
  ...
}
f(4); // implicit - ok!
```



```c++
struct Student {
  explicit Node(int data): data{data}, next{nullptr} {
    
  }
}
```



## Destructor

- is called when an object is destroyed 
  - stack-allocated: goes outof scope 
  - heap-allocated-delete
- every class has its own dtor

Sequence of steps when object destroyed

1. the destructor body runs
2. dtor are invoked for fields that are objects - reverse order
3. space is deallocated 

Why do you need a dtor?

- dynamic memory

```c++
struct Node{
  ...
  Node() {delete next;}
  ...
}
```



Recap:

Destructors 

- method called destructor runs when an object is destroyed 
  - stack allocated - out of scope
  - heap allocated - `delete`
- form 

```c++
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

```c++
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



```c++
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

```c++
Student billy {30,45,50}; // ctor
Student jane = billy; // copy ctor - initializing from a copy
Student joey; // ctor
joey = billy; // copy assignment - after initialization
```

- classes have a default copy assignment operator
  - does a shallow copy
- when do you need to write your own copy-assign operator?
  - dynamic memory

```c++
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

```c++
Node n;
n = n; // self-referencial comparisions
	   // end up deleting this.next other.next
	   // bad - deleted data
```



While writing operator, always check for self-assignment

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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
   - converse not true, normal member functions can still access static members/fields

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