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

``` c++
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

``` c++
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

3. the MIL can be more efficient then initializing in the body of the ctor

   - for fields that are objects and ***not*** in the MIL, they will get default-constructed(and the npotentially initialized again in the ctor body)

4. ***EMBRACE*** MIL, use it!

5. What happens if a field is initialized both in the class and MIL?

   - MIL wins
   - ie. initialization in MIL takes precedence over field init

   eg.

   ``` c++
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

``` c++
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

``` c++
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

``` c++
struct Node {
  ...
  Node(int data): data{data}, next{nullptr} {
    
  }
};
```

- single -argument ctors can lead to implicit behavior

eg

``` c++
Node n(4); // OK - calls ctor
Node n = 4; // Compiler: hmm... - maybe you want a Node? implicit conversion from int -> Node
int f(Node n) {
  ...
}
f(4); // implicit - ok!
```



``` c++
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



``` c++
struct Node{
  ...
  Node() {delete next;}
  ...
}
```

