```c++
class List {
  struct Node;
  Node *theList;
public:
  class Iterator {
    Node *p;
  public:
    explicit Iterator (Node *p):p{p} {}
  };
  Iterator begin() {...}
  Iterator end() {...}
}; // end List
```



**Recall** : Encapsulation and Friends

**Issues**:

client can do this

`auto it = List::Iterator(nullptr)`

- Iterators should not be able to created directly (as shown above)
  - Cliend should have to call List.begin() and List.end() to get an Iterator. 
  - breaks encapsulation (to allow clients to create Iterators directly)
- In making Iterator (and its ctor) visible to list. It's visible and accessible to the client as well
- List needs to be able to create iterators - but nobody else 
  - ie. Iterator ctor is public
  - "special access" called "friend" (private data / stuff)
- fix? Limit ctor so that only List can use it



eg: *using friend*

```c++
class List {
  ...
  public:
  	class Iterator {
      Node *p; // private by default
      expliit Iterator (Node *p);
      ...
      public:
      	firend class List;
  	};
};
```



so: friends have unrestricted access. List can now create iterators

- client can **only** check Iterators by calling List.begin() and List.end() (because these return Iterators)

so, guidelines for friends:

- with classes, we need as few `friend`s as possible
  - friends violate encapsulation 


- in general - keep friends private
- ***but*** what if you want to give a client access to data?
  - create `public` methods to provide access



ie. accessor methods (return data to clients) ("getters")

```c++
class Vec {
  int x, y;
public:
  int getx() const {return x;} // accessor methods
  int gety() const {return y;}
};
// getters
```



mutator methods - change data in fields (setters)

```c++
calss Vec {
  int x, y;
public:
  int getx() {return x}; // accessors
  int gety() {return y;}
  void setx(int newx) {this.x=newx}; // mutators
  void sety(int newy) {this.y=newy};
}
```



So, what's the point?

1. Separates implementation from public interface/methods
   - protects your external interface (ie. public methods)
   - can change underlying implementation w/o affecting clients
2. validation on data in mutaters (bonds-checking) 
   - reject invalid input in mutaters



So, what's the impact of changes to Vec?

- fields are private - standalone functions cannot access fields directly



eg: what about operator << ?

2. Possibilities

   1. use accessor methods in operator <<

      ```c++
      ostream & operator << (ostream & out, const Vec &v) {
        return (out << v.x << " " << v.y);
      } // BAD. breakes because x and y are private
      ```

      operators previously accessed fields (Vec) that are now private

      sol'ns 

      1. ```c++
         ostream & operator << (ostream & out, const Vec &v) {
           return (out << v.getx() << " " << v.gety());
         } // OK. accessors
         ```

         this works fine if you have accessor methods 

         some classes need these accessors

         ​	other classes, accessors don't make sense (eg: List - `getNode()  // ???`)

      2. can also define operator `<<` as a friend function

      ```c++
      // vec.h
      class Vec {
        ...
      public:
        ...
        friend std::ostream & operator << (std::ostream &out, const Vec &x)
      };
      // new, operator access Vec's private fields
      ostream & operator << (ostream &out, const Vec &v) {
        return (out << v.x << " " << v.y);
      }
      ```

very common - friend functions with operators (standalone functions)



**Tools**  Make:

**recall separate compilation**

```shell
g++14 -c list.cc
g++14 -c node.cc
g++14 -c iter.cc
g++14 -c main.cc  # compilation - produces object files (.o)

g++14 list.o node.o iter.o main.o -o myprogram # Linking - produces executable
```



### Reasons for Sep compilation ?

1. compile in any order, dont worry about dependencies
2. only build what's changed (efficient)
   - ie. single file if needed -> how to track this? 
   - in order to track, automate this process 
     - automate compiling
     - keep track of what's changed and only build that

Create a ***Makefile***  (build config file for make) - command-line tool

- one Makefile per project - text file (named Makefile) 
- Makefile  in the same dir as source

```shell
# comment - builds myprogram
myprogram: main.o list.o node.o iter.o
	g++-5 -std=c++14 main.o list.o iter.o node.o -o myprogram
# dependencies
main.o: main.cc list.h
	g++-5 -std=c++14 -c main.cc
list.o: list.cc list.h node.h
	g++-5 -std=c++14 -c list.cc
node.o: node.cc node.h list.h
	g++-5 -std=c++14 -c node.cc
iter.o: iter.cc list.h
	g++-5 -std=c++14 -c iter.cc
```

usage: 

```shell
$ make main.o
```

go into Makefile, find main.o, if either main.cc or list.h changed, rebuild it

``` shell
$ make target
```

build a specific target (eg. main.o, debug, release)

you can add other targets:

```shell
.PHONY: clean
clean:
	rm *.o myprogram
```

``` shell
$ make clean
$ make
```

remove all temp files and rebuild everything



automation - variables

in Makefile:

```shell
CXX = g++-5
CXXFLAGS = -std=c++14 -Wall
...
iter.o: oter.cc iter.h
	${CXX} ${CXFLAGS} -c iter.cc
```

`g++ -W` flag-"warnings"

Can rewrite as 

`iter.o: iter.cc iter.h`



the biggest problem with Makefiles is tracking dependencies 

- however, g++ can figure out dependencies:
- `$ g++14 -MMD -c iter.cc` 
  - produces iter.o (object file) 
  - and iter.d (dependency file)

```
// iter.d
iter.o: iter.cc iter.h
```



eg. that includes generated dependency files 

``` shell
CXX = g++-5
CXXFLAGS = -std=c++14 -Wall
OBJECTS = main.o list.o iter.o node.o
DEPENDS = ${OBJECTS: .o=.d}
```



# System Modeling 

- designing OO systems
- OO programming - all about abstractions; and relationships between abstractions
- helpful to describe relationships, both for design and implementation
- standard: UML (unified modelling language)