## Invariants(recap)

```c++
struct Node{
  int data;
  Node *next;
  //...
  ~Node(){delete next;}
};
```

```c++
int main(){
  Node n1{1, new Node{2, nullptr}};
  Node n2{3, nullptr};
  Node n3{4, &n2};
  //...
  //m3 goes out of scope - destructor 
  //						-delete n2
  //undefined behaviour (segfaults/crashes)
}
```

- issue?
  - nodes allocated on stack
  - n3 — out of scope?
    - dtor — delete n2 — deleting stack - allocated memory
      - undefined behaviour
      - crash!
- we assumed that next would always point to valid mempry on the heap(or nullptr)
  - client tried to use with stack - allocated memory
- invariants — conditions that must be true for our class to work properly/ as expected
- issue? we can't force client to respect our assumptions
  - cannot enforce invariants
  - e.g. stack
    - invariant — last item on the stack is the first item off the stack
    - if a client has direct access to underlying data, puts our invariant at risk

## Encapsulation

- "data hiding"

- we want our client to treat our class as a "black box"

  - should not need to know implementation details
  - client can **only** manipulate our class through provided(public) methods

- "coornerstones" of object-o languages

  ```c++
  struct Vec{
    Vec(int x,y);	//by default methhods are public
  private:		//anything following private cannot be seen
    int x,y;		//   outside of the class
    //...
  public:
    Vec operator+(const Vec &other);	//anything following public 
    									//is accessible to all
    //...
  }
  ```

- By **default**, all members of a struct are public

  - I don't like this — against principles of encapsulation
    - I would prefer things to be private by default

## Class keyword

- identical to a struct <u>except</u> members are <u>private</u> by default

- struct — data only (no methods)

  class — complex (methods)

  ```c++
  class Vec{
    int x,y;	//private
  public:
    Vec(int x,y);
    Vec operator+(const Vec &other);
    //...
  };
  ```

### Recall: Node class, invariant problem

- fix using encapsulation

- write a wrapper class, named list, manages all access to Nodes

  ```c++
  // list.h
  class List{
    struct Node;	//private nested class, accessible only to List
    Node *theList = nullptr;
  public:
    void addToFront(int n);
    int ith(int i);
    ~List();
  }
  ```

  ```c++
  // list.cc
  struct List::Node{
    int data;
    Node *next;
    Node(int data, Node *next):data{data}, next{next}{}
    ~Node{delete next;}
  };

  List::~List(){
    delete theList;
  }

  void List::addToFront(int n){
    theList = new Node{n, theList};
  }

  int List::ith(int i){
    Node *cur = theList;
    for(int j = 0; j < i && cur; ++j)
    {
      cur = cur->next;
    }
    return cur->data;
  }
  ```

- only List can manipulate Nodes, we can guarantee our invariant(i.e. next is always nullptr or a valid pointer to the heap)

  ```c++
  List lst; 	//n elements
  for(int i = 0; i < n; ++i)
  {
    cout << lst.ith(i) << endl;
  }
  ```

  - linear? O(n^2)
  - vary inefficient (vs using a pointer to walk the list)

## SE Topic: Design Patterns

- "Design Patterns" book

- problems recur — are often repeated

  - architectural / design level problems
  - recommended "best practices" for dealing with common problems

- Iterator pattern — designed to solve our exact problem

  ​			     — class manages access to another class

- an iterator is an abstraction of a pointer (in our case, we want it to behave like a pointer into our list)

  - lets us walk the list without exposing pointers

    ```c++
    // list.h
    class List{
      struct Node;
      Node *theList;
      
    public:
      class Iterator{	//public, nested class
    	Node *p;
    public:
      	explicit Iterator(Node *p):p{p}{}
      	int &operator*(){
          return p->data;
     	}
      	Iterator &operator++(){
          p = p->next;
          return *this;
      	}
        bool operator==(const Iterator &other)const{
          return p == other.p;
        }
        bool operator!=(const Iterator &other)const{
          return !(*this == other);
        }
      };
      
      Iterator List::begin(){return Iterator{theList};}
      Iterator List::end(){return Iterator(nullptr);}
    };
    ```

    ```c++
    int main(){
      List lst;
      lst.addToFront(1);
      lst.addToFront(2);
      lst.addToFront(3);
      
      for(List::Iterator it = lst.begin(); it!=lst.end; ++it)
      {
        cout << *it << endl;	//3 2 1
        *it = *it + 2;
      }
    }
    ```

## Automatic type deduction

- compiler infer the type — auto keyword

  ```c++
  for(auto it = lst.begin(); it != lst.end(); ++it)
  {
    //...
  }
  auto i = 5;
  auto s = "string";
  ```

## Range-based for loop

```c++
for(auto n:lst){
  cout << n << endl;
}
```

- Range-based for loops are available in any class c, that:
  1. has methods `begin()` and `end()` that return iterators
  2. the Iterators support
     - operator ++ (prefix)
     - operator !=
     - unary operation *

## Encapsulation(cont'd)

- one (small) criticism of Iterator

  - client can actually create an Iterator
  - `auto it = List::Iterator(nullptr);`

- violates encapsulation

  - we want client to use begin() and end()

- issue is the Iterator's constructor is public

- if you make the ctor private, List can't create Iterators either!

- what do we do?

  - List should have privileged status with Iterator

  - friend — List is a friend of Iterator

    ​	    — keyword that says your friends have access to your private data