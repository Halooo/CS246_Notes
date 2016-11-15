# NVI Idiom (Non-virtual interface)

A public virtual method is really 2 things

1. a public interface for the client
2. an interface for subclasses - override to specializing behavior

These 2 things should not be tied

eg: we want to change implementation later without changing public methods



**NVI Idiom States:**

1. all public methods should be non-virtual - client interface (will not change)
2. all virtual methods should be private/protected - specializing derived classes
   - exception: dtor is still public (even if virtual)
   - want dtor to be virtual, but needs to be public



eg: Digital media - Non-virtual
Bad code. Mixing public and virtual method together
``` c++
class DigitalMedia {
public:
  virtual void play() = 0;
  virtual ~DigitalMedia();
};
```
Good code
``` c++
class DigitalMedia {
public:
  void play() {
    doPlay();
  }
  virtual ~DdigitalMedia(); // public interface
private:
  virtual void doPlay()=0; // private and virtual
};
```

If we need extra control (later on) we can claim it
eg: 
-  can add code before/after doPlay to check copyright, update play count
-  we can add more "hooks" for derived classes by. eg: adding other virtual methods to showCoverArt
-  all without changing the public interface


It's much easier to design early than to try and take control later

Basically: NVI puts every virtual methods inside a non-virtual wrapper



# STL Maps
aka hash tables

store a key/value pair in data structure

eg: "arrays" that store string/int

```c++
#include <map>
using namespace std;
map<string, int> m;
```

```c++
m["abc"] = 1;
m["def"] = 4;
cout << m["abc"] << end; // 1
cout << m["ghi"] << emd; // 0
// if the key doesnt exist, compiler will default-construct a pair and return value
```

```c++
if(m.count("def")) {
  cout << m["def"] << endl;
}
m.erase("abc");
for (auto &p:m) {
  cout << p.first << p.second << endl; // p.first is key, p.second is value
}
```

p's type here is `std::pair<string, int>`

- defined in `<utility>`


# Inheritance & copy/move operators

``` c++
class Book {
public:
  //defines copy/move ctor, copy/move assignments
}

class Text:public Book {
  string topic;
public:
  // does not define copy/move ctor or copy/move assignments
}
```

```c++
Text t{"Algorithms", "CLRS", 500, "CS"};
Text t2=t1; // copy stor - but no copy ctor for Text
```

What happens?

- copy init calls Book's copy ctor. and then goes field-by-field for the Text part. ie. default behavior. 
- some is true for other compiler - supplied methods



```c++
Text:: Text(const Text &other): Book{other}, topic{other.topic} {}; // copy ctor
Text &Text::operator=(const Text &other) { //copyt assignment
  Book::operator=(other);
  topic = other.topic;
  return *this;
}
Text::Text(Text &&other): Book{std::move(other)}, topic{std::move(other.topic)} {} //move ctor7
Text &Text::operator=(const Text&& other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic0);
  return *this;
}
```

These mimic default behavior(ie shallow copy)

Note: even though other "points" to an r-value, other is actually a l-value

- to fore move ctor/assign, we use `std::move` to fore other
- to be treated as an r-value - ie. move versions are used

Now consider

```c++
Text t1 {...};
Text t2 {...};
Book *pt1 = &t1;
Book *pt2 = &t2;
```

what happens if we do: `*pt1 = *pt2;` copy assignment - Book version runs

So, Book::operator = runs;

- problem: topic is nwer assigned - BAD

Partial assignment - base class ptr means base class operator runs

- ie fields not assigned

Fix - make operator= virtual

```c++
class Book {
public:
  virtual Books & operator=(const Book&other) {}
}
class Text:public Book{
public:
  Text &operator=(const Book) override {}
}
```

Note: Text can return a subtype of Book - allowed

but parameters need to be the same (ie. Book & for both)

(if not the same wont compile)

by the is-a principle. b/c a Text is a Book, you can do this

assign of a Book object to a Text object can be done

``` c++
Text t {...};
Book b {...};
Text *pt = &t;
Book *pb = &b;
*pt = *pb
```

Technically works, aka compiles

- but not that useful - init Text from a Book, cannot init topic

Also implies:

```c++
Comic c {...};
Comic *pc = &c;
*pt = *pc; // worse. cannot init Text::topic field also ignoring Commic::hero
```

So, if operator= is non-virtual, then we get *partial assignment* when assign from base class pointers; if it's *virtual*, we get *mixed assignemnt*

### Recommendation:

all superclasses should be abstract



```c++
class AbstractBook {
  string title, author;
  int numPages;
protected:
  AbstractBook &operator=(const AbstractBook &other);
public:
  AbstractBook(...);
  virtual ~AbstractBook()=0;
};
AbstactBook::~AbstractBook(){}
```

- protected operator= prevents assignment through base class pointers from compiling 
  - derived classes can still invoke



```c++
class NormalBook:public AbstractBook {
public: 
  NormalBook();
  ~NormalBook();
NormalBook& operator=(const NormalBook& other) {
  AbstractBook::operatpr=(other);
  return this;
}
}
```

This design prevents both partial and mixed dessign

- can still uses base class pointers for derived classes

