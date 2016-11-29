# Casting

Forcing an object to be treated as another object

In C:

```c
Node n;
int *ip = (int *)&n; //cast Node* to an int*
// convert "blindly" - "trust me!" ;)
```

 

C-style cast should be avoided in C++

In general, you want to avoid casts

"Casts are ***so*** bad, c++ gives you **4** ways to do them."

- if you *must* cast, use c++ casting mechanisms
- 4 types - very specific, describe the intent of the cast
- deliberate - compiler wont infer a cast - forces you to type something



- eg: 1. static-cast - "reasonable and suitable" double -> int

- ```c++
  void f(double x);
  void f(int y);
  double d = 30;
  f(d); // calls 1
  f(static_cast<int>(d)); //calls 2
  ```

- eg: supclass ptr -> subclass ptr

- ```c++
  Book *bp = new Text{...}
  Text *tp = static_cast<Text *>(b);
  ```

  - legitimate cast in this example
  - you are taking responsibility - compiler not checking validity - telling the compiler - "trust me!"

- eg: 2. reinterpret - cast - strange, unusual

- ```c++
  Student s;
  Turtle *t = reinterpret_cast<Turtle *>(&s);
  ```

  - this example makes no sense - compiler does no checking uses cast to determine how to interpret memory.
  - Reinterpret 
    - telling the compiler - "I trust you to do something"
    - not  guaranteed to make sense
    - you have to ensure that its a reasonable cast
    - very rare to use this

- eg: modifying a private variable

- ```c++
  #include <iostream>
  using namespace std;
  class C {
    int x; //private
  public:
    explicit C(int xval):x{xval}{}
    int getx() {return x;}
  };

  class RogueC {
  public:
    int x;
  };

  int main() {
    C c(10);
    cout << c.getx() << endl;  // 10
    RogueC *r = reinterpret_cast<RogueC *>(&c);
    r->x = 20;
    cout << c.getx() << endl; // 20
  }
  ```

  - in other words, its all just memory - for a reinterpret cast

- eg: 2D arrays on the heap

  - specify memory as a big block
  - 2D array, need to figure out how to index into the block
  - example code in course repo

- eg: 3. const cast

  - remove "const-ness" - const-poisoning 

  ```c++
  void g(int *p); // g makes no promises about p - p may be changed
  void f(const int *p) { // f promises to not change p
    g(p); // D: (compiler unhappy) error
  }
  ```

  This will not compile since g() has a weaker commitment than f()

  ie. g doesnt promise to not change p

  **But**, I really want to ... I know that g() will not change p ("trust me!")

  the designer forgot  const

  ``` c++
  void g(int *p);
  void f(const int *p) {
    g(const_cast <int*p>(p));
  }
  ```

- eg: 4. dynamic cast

  - similar to a static cast(ie "reasonable")
  - "is it safe to cast x to y?"

  ```c++
  Book *bp = getABook(); // factory method
  static_cast<Text *>(bp)->getTopic();
  // Q: is this safe?
  // depends on whether its actually a Text
  // A: not safe!
  ```

  apply dynamic cast - try and see if it succeeds

  ```c++
  Book *bp = getABook();
  Text *tp = dynamic_cast<Text*>(bp);
  ```

  if the cast works, (it is a Text), tp will point to a Text object

  if the cast fails, (it is **not** a Text), tp will equal `nullptr`

  ```c++
  if(tp) cout << tp->getTopic();
  else cout << "not a text";
  cout << endl;
  ```

  so, why use static_cast sometime? 

  static_cast is free, cost no time. dynamic_cast costs time



so, there are operators on raw pointers

- we also have versions for smart-pointers
  - static_pointer_cast
  - const_pointer_cast
  - dynamic_pointer_cast
- there is no reinterpret_pointer_cast
- use exactly the same way, but the operate on shared_ptrs (and return shared_ptrs)

You can use dynamic casting to make decisions at runtime based on object types (RTTi - runtime type info)

eg: 

```c++
void whatIsIt(shared_ptr<Book> b {
  if (dynamic_ptr_cast<Text>(b)) cout << "Text";
  else if (dynamic_ptr_cast<Comic>(b)) cout << "Comic";
  else cout << "Book";
}
```

Dynamic casting only works on class with at least one virtual method

- why? will explain later
- in practice, not a problem- virtual dtor

Code like this is very tightly coupled to book hierarchy

- eg: add a new type of book (eg: cookbook) is difficult
- better ways - virtual methods, visitor pattern



Dynamic casting also works with references 

```c++
Text t {};
Book &b = t;
...
Text &t2 = dynamic_cast<Text &> b;
```

If b points to a Text - this works fine

If b does not point to a Text - cast fails and throws exception (bad cast exception)



We can use dynamic reference casting to solve the polymorphic assignment 

``` c++
Text & Text::operator=(const Book &other) {
  // issue - Book doesnt have a topic field
  // use cast to see if other is a Text with all fields
  Text & textother = dynamic_cast<Text &> other;
  if (this == &textother) return *this;
  Book::operator=(other);
  topic = textother.topic;
  return *this;
}
```

If we wanted to handle all cases - we could dynamic_cast to Comic... Book... and avoid an exception



