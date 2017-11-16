# System Modelling
Building an OO system identifying abstractions of real world things and relationships between those abstractions.

So we want a way to quickly model and represent these relationships, which can be a blueprint or just a quickly readable representation of our program.

## Unified Modelling Language (UML)

|name of the class| Vec |
|---|---|
|fields (optional)|-x: integer|
||-y: integer|
||// the "-" means that it's private|
|methods (optional)|+ operator+ : Vec|
||+ getX : integer|

Visibility:
  - `-` stands for private
  - `+` stands for public

## "Owns-a" relationship

```c++
class Vec {
  int x, y;
  public:
    Vec(int x, int y) : x{x}, y{y} {}
};

class Basis {
  Vec v1, v2;
  // add this to get rid of error
  Basis() : v1{1,2}, v2{3,7} {}
};

Basis b; // error!
```

We try to default construct `b` with Basis's built in default ctor which just default constructs all fields that are objects, namely `v1` and `v2`, but `Vec` doesn't have a default ctor.

Embedding an object inside another object. This kind of a relationship is called a composision. A composition relationship is also another broader type of a relationship called an "owns-a" relationship.

If `A` owns `B`, then:
  - If `A` dies so does `B`
  - `B` has no identity outside of `A`, if you're talking about `B`, it's in the context of `A`
  - If you copy `A`, then you're also copying `B` (deep copy)

Example: A car owns 4 wheels, they are a part of that car. If the car is destroyed, the wheels are too. If you copy a car, you copy it's wheels as well.

Implementation of an "owns-a" relationship is usually done through compilation of classes.

Modelling:

**compositional**

![uml-relation-pointers]()


## "has-a" relationship
**aggregation**

Compare the car to music player's playlists. The car owned it's components, what about a playlist? Deleting a playlist doesn't delete those songs. This is a "has-a" relationship. (aggregation)

If `A` has `B`, then typically:
  - If `A` dies, `B` still exists
  - `B` has existance apart from its associated `A`
  - If `A` is copied, `B` is not (shallow copy) thus copies of `A` share the some `B`

Typical implementation:
  You have pointers to the things you "have"

## Inheritance

We have seen owns-a and has-a relationships. But let's consider the relationships in the following example:

```c++
class Book {
  string title, author;
  int numPages;
  public:
    Book(...) { ... }
};

class Comic {
  string title, author, hero;
  int numPages;
  public:
    Comic(...) { ... }
};

class Text {
  string title, author, subject;
  int numPages;
  public:
    Text(...) { ... }
};
```

This is alright, but does it really show the relationship between these types?

Really - A comic is a book and so is a text. Furthermore, how do we use an array (collection) of our library, if it needs to hold these different types.

```c++
union BookTypes { Book *b, comic *c, text *t };
BookTypes myBooks[20];

// This is BAD
```

We can have an array of void pointers and cast our book/comic/text ptrs to void ptrs, and cast back to use.

Rather: We know textbooks are books, and comics are books. A comic `is-a` book, a text `is-a` book. So we want to model an `is-a` relationship. So we can employ the oop concept of Inheritance to solve this problem.

```c++
// base class
class Book {
  string title, author;
  int numPages;
  public:
    Book(string t, string a, int n)
    : title{t}, author{a}, numpages{n} {}
}

// derived class
class Comic : public Book {
  string hero;
  public:
    Comic(...) { ... }
};

// derived class
class Text : public Book {
  string subject;
  public:
    Text(...) { ... }
};
```

  - Derived classes inherit their fields from the base class (`title`, `author`, `numPages`)
  - Any method we can call on a Book, we can call on a `Comic` or a `Text`

Now, consider this ctor for the `Comic` class

```c++
Comic::Comic(string title, string author, int numPages, string hero)
: title{title}, author{author}, numPages{numPages}, hero{hero} {}
```

This fails for the following reasons:
  - `title`, `author` and `numPages` are private to `Book`, we can't access them in `Comic`
  - When objects are created:
    1. Space is allocated
    2. Base class component is initialized
    3. Fields are initialized
    4. Ctor body runs
  - But `Book` has no default ctor!

This works.

```c++
Comic::Comic(string title, string author, int numPages, string hero)
: Book{title, author, numPages}, hero{hero} {}
```

If the base class has no default ctor, you must specify how to construct it in the MIL

There are good reasons to keep the fields hidden of the base class from the subclass, but if you want to reveal these, you can, by specifying them as protected.

```c++
class Book {
  string title, author;
  int numPages;

  protected:
    void addAuthor(string a) { ... }

  public:
    Book(string t, string a, int n)
    : title{t}, author{a}, numPages{n} {}
};
```

Derived classes can access protected fields

```c++
class Text : public Book {
  string topic;
  
  public:
    void addAuthor(string a) {
      Book::addAuthor(", " + a);
    }
};
```

But why put `addAuthor` in the derived class?

If we want to manipulate it, it makes sense to provide a method in the class that contains that fields.

In `C++`, an `is-a` relation is implemented using public inheritance.

```c++
class Text : public Book;

text t{ ... };
t.getAuthor() // okay

class Text : protected Book;
t.getAuthor() // error, `getAuthor` is protected
```

Now, let's consider a method to tell us if a book (of any type) is heavy

```c++
class Book {
  ...
  ...

  protected:
    ...
    ...

  public:
    bool isItHeavy() const { return numPages > 200; }
    ...
    ...
};

class Comic : public Book {
  ...
  ...

  public:
    bool isItHeavy() const { return numPages > 50; }
    ...
    ...
};

class Text : public Book {
  ...
  ...

  public:
    bool isItHeavy() const { return numPages > 500; }
    ...
    ...
};

int main() {
  Book b { "A book", "author", 78 };
  Comic c { "A comic", "author", 78, "hero" };

  cout << b.isItHeavy(); // false
  cout << c.isItHeavy(); // true
}
```

What about this?

```c++
Book b = Comic{ "A title", "Author", 78, "hero" };
cout << b.isItHeavy() << endl; // false
```

Here, `b` only takes `Book` part of the `Comic`, `hero` is ignored using book's `move` ctor

Compiler tries to fit `Comic` into a `Book`, it slices it, ignoring the comic's field

What happens if we access a comic object through a book ptr?

```c++
Comic c { "A title", "Author", 78, "hero" };
Book *pb = &c;
```

Slicing is unnecessary, and if slicing is unnecessary, we could validly call any comic functions.

```c++
cout << pb->isItHeavy() << endl; // false
```

The compiler only knows `pb` is of type Book *, it doesn't know, it only points at a comic object. As such the only thing it can do with the provided knowledge is call the corresponding types fn.

How to make a `Comic` object behave like a `Comic` object even when pointed to by a `Book *`?

## Polymorphism

If there is an inheritance relationship between two classes, an instance of the subclass can be used anywhere an instance of the superclass can be used.

This means that these are valid:

```c++
// B inherits from A
B b{1, 2};
A a = b;

A* a = new B{3, 4};

void foo(A a);
void foo2(A& a);

foo(b);
foo2(b);
```

However, a B object is larger than an A object
  - This means that any time we force a B object into an A object, it doesn't fit and the object will be sliced; only the A part of the object will be copied or moved or considered valid in case of a pointer or a reference.
  - The slicing is called ***object coercion***

## `override` and `virtual`

How to make a comic object act like a comic object regardless of the pointer or reference type it is accessed through?

- Solution is to declare the `isItHeavy` method as `virtual`

With `virtual` methods, we choose which class's method to run, based on the actual type of the object, not based on the type of whatever we're using to access that object.

### Example

```c++
// My book collections
Book *MyBooks[20];
...
for (int i = 0; i < 20; ++i) {
  cout << MyBooks[i]->isItHeavy;
}
```

This code for iterating the array accomodates multiple types under one abstraction(Book). This is called Polymorphism, "many forms"

### Notes

- This way we have been able to write functions that take in `istream&` and pass them an `ifstream&`, because an `ifstream` is an `istream`
- Never use arrays of objects polymorphically. If you need polymorphic arrays, use an array of ptrs.

## Destructor Revisited

```c++
class X{
  int *x;
  public:
  X(int x) : X{new int {x}} {}
  ~X() { delete x; }
}

class Y{
  int *y;
  public:
  Y(int x, int y) : X{new int {x}}, y{new int{y}} {}
  ~Y() { delete y; }
}

Y *py = new Y{1,2,3};
delete py;  // this is fine
X *px - new Y{1,2};
delete px; // leak the y position of the object
```

How can we ensure deletion through a base class ptr calls the right destructor? Make it virtual.

__Always declare your destructor virtual when you have inheritance.

- If a class `Y` is not meant to ever be derived, declare `final` - `class Y final : public X { ... }`

## Pure Virtual and abstract classes

```c++
class Student {
  protected:
    int numCourses;
  public:
    // declares Student Fees as pure virtual
    virtual int fees()=0;
};

class Regular final : public Student {
  public:
    int fees() override {
      // calc fees for reg stud
    }
};

class Coop final : public Student {
  public:
    int fees() override {
      // calc fees for coop stud
    }
};
```

What should `Student::Fees` do?

Nothing. It makes no sense to create a student object, all students are either regular or coops. So we explicitly give `Student::fees` no implementation by setting it equal to 0, declaring it pure virtual.

```c++
Student s; // error
Student *reg = new Regular{ ... }; // okay
```

Any class with at least 1 pure virtual method, cannot be instantiated as an object. Such a class is called an **Abstract Class**

Subclasses of an abstract class must implement the pure virtual methods, or else they're also abstract classes.

A class that is not an abstract class is called a **Concrete Class**.

In UML, an abstract class and virtual methods are denoted by *italicizing* their names.

## Copy and Move ctors/ops with inheritance

```c++
class Book {
  public:
    // defines copy/move fns
};
```

If `Book`'s Copy Assignment Operator isn't `virtual`

```c++ 
Text& operator=(const Text &o) {
  Book::operator=(o);
  topic = o.topic;
  return *this;
}
```

But now if we do

```c++
Text t1 {...}, t2 {...};
Book *pb1 = &t1, *pb2 = &t2, *pb1 = *pb2; //!!Partial assignment, calls Book::operator
```

But if we make `Book`'s Copy Assignment Operator `virtual`, then to overide it, `Text`'s CAO type signature must match.

```c++
Text& Tex::operator=(const Book &o) {
  Book::operator=(o);
  topic = o.topic;
  return *this;
}

But this allows the following to compile

```c++
Text t{...};
Book b{...}; // Doesn't have topic
t = b; // runs Text's CAO and crashes
```
**Suggestion**: Any superclasses you inherit from should be made abstract.

```c++
class AbstractBook {
  int numPages;
  String title, author;
  protected:
    Abstract Book& operator=(const Abstract Book &o) ...
  public: // Since CAO is protected, client can't assign through base class pointers.
    Abstract Book(...);
    Virtual ~AbstractBook() = 0;
}

**Note** Even though we declared `~AbstractBook` pure virtual, we still need to provide an implementation or else our program won't link.

```c++
// AbstractBook.cc
AbstractBook::~AbstractBook() { }
```
For other classes, headers/code would be similar, adapt accordingly. This design prevents partial assignment and mixed assignments.

**Don't make concrete superclasses, make them abstract**

# Templates

Note: We are **not** covering the concept of templates in entirety, just some stuff.

```c++
class List {
  struct Node;
  Node *head;
  ...
  ...
};

struct List::Node {
  int data;
  Node *next;
}
```

What if we want to store something other than `int`s? Do we have to write a whole new `class`? Well, a template class is a class that is parametized by a type.

## Example: A stack

```c++
template <typename T> class Stack {
  int size;
  int capacity;
  T *data;
public:
  stack() ...
  void push(T *) ...
  T top() const ...
  void pop() ...
}
```

## Template for our List

```c++
template <typename T> class List {
  struct Node {
    T data;
    Node *next;
  }

  Node *head;

public:
  class Iterator {
    Node *p;
    Iterator(Node *p) : p{p} {}
    T& operator*() { return p->data; }
    bool operator!=(const Iterator &o) { return p != o.p }
    Iterator& operator++() { p = p->next; return *this; }
    friend class List;
  };

  Iterator begin() { return Iterator{head}; }
  Iterator end() { return Iterator(nullptr); }
}
```

But how do I use this template?

```c++
List <int> l1;
List <List <int> > l2;
List <List <List <int> > > l3;
// need spaces around the closing angular braces

l1.addToFront(3);
l2.addToFront(l1);
l3.addToFront(l2);

for (auto &n : l1) {
  cout << n << endl;
}

// or this

for (auto &l : l2) {
  for (auto n: l) {
    cout << n << endl;
  }
}
```

How do templates work? We make the compiler do it.

Roughly speaking, the compiler makes a separate class each time a templated class is parameterized with a new type (at the machine code level (not really))

## The Standard Template Library

It is a large collection of template classes provided as part std c++

### Dynamic size arrays - Vector

```c++
#include <vector>

using namespace std;

vector <int> v{4, 5};
// creates the array [4, 5]
// calls the list initialisation ctor

vector <int> v2(4, 5);
// This calls vector(int, T)
// v(4, 5) creates [5, 5, 5, 5]

v.emplace_back(6); // 4,5,6
v.emplace_back(7); // 4,5,6,7

for (int i = 0; i < v.size(); ++i) {
  cout << v[i] << endl;
}
```

Vector implements the iterator pattern

```c++
for (vector<int>::iterator it = v.begin(); it != v.end(); ++i) {
  cout << *it << endl;
}

// or this

for (auto n : v) {
  cout << *it << endl;
}

// or the reverse iterator

for (vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++it) {
  cout << *it << endl;
}
```

`v.pop_back()` removes the last element of `v`

Vectors are generated to be implemented as an array, so from now on, if you use a dynamic array, use vector. There is no more reason to use the array forms of `new[]` and `delete[]`

Other vector operations are based on iterators such as `erase`, which erases the element pointed to by the iterator

```c++
auto it = v.erase(v.begin());
// erases the item as index 0, returns the iterator to the first item after the erase

it = v.erase(v.begin() + 3);
// erases the item at the 3rd index

it = v.erase(it);
// erases the item pointed to by it

it = v.erase(v.end() - 1);
// erases the last item on our list
```

`v[i]` returns the `ith` element of v. If you go out of bounds, vector doesn't check for you - undefined behaviour.

We can instead use `v.at(i)`, which is a checked version.

But what does it do when you go out of bounds? What should happen?

Problem: Vector code can detect the error, but doesn't know what to do about it. The client knows what they do if it occurs, but can't detect it. Error recovery, by nature is a non-local problem.

**C Solution:** fn returns a status code or if output eange completely used set a global variable. These are awkward and clunky solutions, which lead to wither bad code or ignored errors.

**C++:** when an error condition arises, the function can raise an exception. What happens when an exception is raised? The program terminates by default. However the client can catch the exception and handle it.

By default the program terminates after unwinding the call stack. But we can write handlers to resolve the exception and deal with them.

`Vector<T>::at` raises an exception of type `std::out_of_range`

```c++
#include <stdexcept>
...
try {
    cout<<v.at(100000)<<endl;
} catch (out_of_range) {
    cerr<<"Index out of range"<<endl;
}
```
Now consider:

```c++
void f(){
    throw std::out_of_range("f");
}

void g(){f();}
void h(){g();}

int main(){
    ...
    try {
        h();
    } catch(out_of_range){
        ...
    }
}
```
What happens? `main` calls `h`, which calls `g`, which calls `f`. `f` throws a `out_of_range` exception. 

Control goes backwards through the call chain (unwinds the stack) until an appropriate handler is found. In this case in `main`, which then handles the exception. 

If no such handler is found, the program crashes. 

What is `out_of_range`? It's a `class`, so the statement `out_of_range("f")` creates an `out_of_range` object using a single string parameter constructor. The string is for including auxilliary information. 

```c++
try {
    ...
} catch(out_of_range ex) {
    cout<<ex.what()<<endl; 
    //Note the similiarites to a parameter declaration.
} 
```

A handler can do part of the recovery job, i.e. execute some corrective code and throw another exception. 

```c++
try {...}
catch(some_error_type s) {
    ...
    throw someOtherError(...);
}

try {...}
catch(some_error_type s) {
    ...
    throw;
}
```
Why `throw` instead of `throw s`? 

The exception could actually be a type of derived class of `some_error_type` in which case we created `s` by slicing that object. If we throw `s`, we're throwing the sliced object. 

If we just say `throw`, we're throwing the original exception we recieved, maintaining its type/behaviour.

---

A handler can act as a catch

```c++
try{...}
catch(...) {//literally means ... here}
...
```
An exception can be anything, not just objects. It can be a POD type if you want. Typically though will be a class you've made to be meaningful, or an appropriate std exception. 

```c++
class BadInput{};
...
try{
    int n;
    if (!(cin>>n)) throw BadInput;
} catch(BadInput &) {
    cerr<<"Input not well-formed\n";
}
```
**Note**: The exception is caught by reference. This prevents the exception from being sliced. So if we had actually caught an object of some derived class of `BadInput`, we could get the actual correct behaviour of that object in our catch. 

So the general rule of thumb is *catch by reference*, *throw by value*.

Some std exceptions
* `length-error` attempting to resize strings/vectors that are too long.
* `bad_alloc` is thrown when new fails.

**NEVER!!** let a destructor `throw`, since when an exception is thrown, only statically allocated objects inside a function being unwound will be destroyed if their destructors throw. Now you have two unhandled exceptions and the program terminates immediately. 

# Design Patterns Continued

## Observer Pattern

Publish-Subscribe model

* One class: the publisher or subject. 
* One or more classes: called subscribers or observers, who receive the data and react. 

Example: Publishers in the form of spreadheet cells, and subscribers in the form of graphs.

Refer to ![Diag 1](/res/ObserverPattern.png)

The abstract class subject has all the common code to all subjects. The abstract class observer holds only the interface common to all observers. 

### Sequence of Method Calls
1. Subject's state is updated
2. `Subject::notifyObservers()` calls each observer's `notify` method.
3. Each observer calls `ConcreteSubject::getState` to get the data and reacts to it. 

Example: Horse races. The Subject publishes the winners, the observers are individual betters. They will declare victory if their horse wins. 

```c++
class Subject{
    Vector<Observer *> observers;
    public:
        Subject();
        void attach(Observer * ob) {
            observer.emplace_back(ob);
        }
        void detach(Observer * ob);
        void notifyObservers() {
            for (auto &ob: observers) {
                ob->notify();
            }
        }
        ~Subject()=0;
};

Subject::~Subject(){}

class Observer {
    public:
        virtual void notify()=0;
        virtual ~Observer(){}
};

class HorseRace : public Subject {
    ifstream in; // Source of data
    String lastWinner;
    public:
        Horserace(String source): in{source}{}
        ~Horserace(){}
        bool runRace(){
            return in>>lastWinner;
        }
        String getState() const {
            return lastWinners;
        }
};

class Bettor : public Observer {
    Horserace * subject;
    String name, myHorse;
    public:
        Bettor(HorseRace *hr, std::string name, std::string horse) :
        subject{hr}, name{name}, myHorse{horse} {
            subject->attach(this);
        }
        void notify() {
            String winner = subject->getState();
            cout<<(winner == myHorse)? "win" : "lose")<<endl;
        }
};
```
Simplifications and Adaptations are possible in certain circumstances. 
* If there is only one subject, you can merge subject and concrete subject. Do not do so lightly as this reduces your ability to generalize in the future.
* If the state is trivial (i.e. being notified is all the information we need), then you can leave out `getState`.
* If a subject and an observer can be the same thing (e.g. cells), you can have them be one class. 

Suppose you want to enhance an Object, e.g. add funcionality or features at runtime. For example a windowing system, we start with a basic window, add a scroll bar, and a menu. We would like to be able to choose these enhancements at runtime.

## Decorator Pattern

![diag2](/res/DecoratorPattern.png)

The `class` component defines the interface: the operations your objects will provide. The class `concreteComponent` implements the interface. The `concreteDecorator` classes inherits from `Decorator`, which inherits from `Component` itself. Ergo all decorators are components and they have components. 

For example, a window with a scrollbar ia a kind of window, and has a pointer to the underlying plain window. A window with a scrollbar and a menu is a window, and has a ptr to a window with a scrollbar. 

All of these inherit from window, so window methods can be called on them *polymorphically*.

![diag3](/res/DecoratorPatternPizza.png)

```c++
class Pizza {
  public:
    virtual float price() const = 0;
    virtual string desc() const = 0;
    virtual ~Pizza();
};

class CrustAndSauce : public Pizza {
  public:
    float price() const override {return 5.99;}
    string desc() const override {return "Pizza";}
};

class Decorator : public Pizza {
  protected: 
    Pizza *p;
    public: 
      Decorator(Pizza *p): p{p}{}
      virtual ~Decorator() {delete p;}
};

class StuffedCrust : public Decorator {
  public:
    StuffedCrust(Pizza *p): Decorator{p}{}
    float price() const override {
      return p->price() + 2.69;
    }
    string desc() const override {
      return p->desc() + "EWith Stuffed Crust";
    }
};

class Topping : public Decorator {
  string theTopping;
  public:
    Topping(Pizza *p, string theTopping): 
      Decorator{p}, theTopping{theTopping}{}

    string desc() const override {
      return p->desc() + "with" + theTopping;
    }  
    float price() const override {
      return p->price() + 1.001;
    }
};

// class DippingSauce has been omitted

//main.cc

Pizza *p1 =  new CrustAndSauce;
p1 = new Topping(p1, "Cheese");
p1 = new Topping(p1, "Ham");
p1 = new StuffedCrust(p1);

cout << p1->desc() <<" "<< p1->price();
cout << endl; 
```
Guiding principle is to program to interfaces, not implementations. 

**Abstract Base Classes** define the interface, work with pointers or references to Abstract Base Classes and call the methods declared in the interface. This way `ConcreteClass` objects can be swapped in and out at will. 

Example: Iterator Pattern

```c++
class AbstractIterator { // could be templated.
  public:
    virtual int operator*()=0;
    virtual bool operator!=(const AbstractIterator &o)=0;
    virtual AbstractIterator & operator++()=0;
    virtual ~AbstractIterator();
};

class List {
  struct Node;
  ...
  public:
    class Iterator : public AbstractIterator {
      ...
    };
};

class Set {
  ...
  public:
    class Iterator : public AbstractIterator {
      ...
    };
};
```
Now we can write functions that operate on `Iterators` that are derived from `AbstractIterator`. In this way we can use the same functions to iterate over a variety of different data structures, without knowing what the underlying structures are.

```c++
template <typename Fn> 
  void forEach(AbstractIterator &start, AbstractIterator &end, Fn f) {
    while (start != end) {
      f(* start);
      ++start;
    }
  }
```
## Factory Method Pattern

Problem: Write a video game with two types of enemies, bullets and turtles. The system randomly spawns Bullets and Turtles, but bullets should become more frequent in later levels. 

![diag 4](/res/FactoryMethodEnemy.png)

Since we never know what enemy is going to spawn next, we can't call constructors directly. We also don't want to hard code the policy since we want it to be customizable and adaptable. 

So we put a `factoryMethod` in `Level` that creates our enemies for us.

```c++
class Level {
  public:
    virtual Enemy *createEnemy()=0;
    //factory method
    ...
};

class NormalLevel : public Level {
  public: 
    Enemy *createEnemy() override {
      // create more turtles
    }
};

class Castle : public Level {
  public:
    Enemy *createEnemy() override {
      // create more bullets
    }
};

//main.cc

Level *l = new NormalLevel;
Enemy *e = l->createEnemy();
```

## Template Method Pattern

We want subclasses to override some behaviour but still have some common behaviour across all our derived classes. 

**Example**: We have red turtles and we have green turtles. We want to draw our turtles. 

```c++
class Turtle {
public:
  void draw() {
    drawHead();
    drawShell(); //virtual
    drawFeet();
  }
private:
  void drawHead() {...}
  void drawFeet() {...}
  virtual void drawShell()=0;
};

class RedTurtle : public Turtle {
  void drawShell() {// draw a red shell}
  ...
};

class GreenTurtle : public Turtle {
  void drawShell() {// draw a green shell}
};

```

Subclasses can't change the way a turtle is drawn, but can specialize drawing skills.

A `public virtual` method is really two things.
- `public`: It is an interface to the client. It indicates the behaviour we're providing to the client, upholds class invariants and pre/post conditions.
- `virtual`: An interface to a subclass. A *hook* for our derived classes to inherit specialized behaviour. 

It's hard to separate these two ideas from each other if they're wrapped up in one method. 
* What if we later decide one of our virtual methods has grown too large and decide to split it into two, we now have to change not only out class code but also our client code. 
* How could we make sure our overriding classes conform to pre/post conditions? What if they change later? Ugly code duplication.

## The Non-Virtual Interface Idiom (NVI)

It says 
* All `public` methods should be non-virtual
* All `virtual` methods should be `private`
* Except obviously the destructor

e.g. Digital Media

```c++
class DigitalMedia {
public:
  virtual void play()=0;
  virtual ~DigitalMedia();
};
```
**NVI**

```c++
class DigitalMedia {
public:
  void Play() {doPlay();}
  virtual ~DigitalMedia() {}
private:
  virtual void doPlay()=0;
};

void DigitalMedia::Play() {
  if (valid_subscription) {
    doPlay();
  }
}
```
Now if we need extra control over `Play()` we can do it.
* We can add before/after code to run around `doPlay()` (also could be specialized).
* Can add more hooks later through more `virtual` method calls.
* All of this without changing the public interface.

It is much easier to keep control over our interface and `virtual` methods than to retake it after the fact.

So the NVI idiom extends the template method pattern saying all virtual methods should be wrapped in a non-virtual method. This is a huge benefit with basically no downside - **Do it**

## STL Maps: For Creating Dicts

e.g. For creating an "array" that can be indexed be an arbitrary type, i.e. a string.

```c++
#include <map>
using namespace std;

map<string, int> m;
m["abc"] = 1; // if key doesn't exist, create it and assign the coresponding value
cout << m["abc"] << endl; //prints 1
cout << m["ghi"] << endl; //prints 0
```

If a key doesn't exist, indexing by that key creates it and default initializes it. Plain Old Datatypes have a default initializing for `map` only. 

```c++
m.erase("abc"); // removes key-value pair
if (m.count("abc")) { 
  // produces the count of that key, 0
}

for (auto &p : m) {
  cout << p.first << "," << p.second << endl;
}
```
Prints in order based on key values. `p`'s type here is `std::pair<string, int>`.

## Visitor Pattern

For implementing double dispatch. Virtual methods are chosen on the actual runtime type of objects. What if we have a function that only operates on two objects?

*ADD LECTURE HERE

----

```c++
class Book { //enemy
public:
  ...
  virtual void accept(BookVisitor &v) {
    v.visit(*this); // beStruckBy
  }
};

class Text : public Book {
public:
  ...
  void accept(BookVisitor &v) override {
    v.visit(*this);
  }
};

class Comic : public Book {
public:
  ...
  void accept(BookVisitor &v) override {
    v.visit(*this);
  }
};

class BookVisitor { //weapon
public:
  virtual void visit(Book &b) = 0; // Strike
  virtual void visit(Text &t) = 0; // Strike
  virtual void visit(Comic &c) = 0; // Strike
};
```
Track how many books of each type we have. Group books by author for `Book`, hero for `Comic` and topic for `Text`.

We use a `map<string, int>`. We could add a `virtual` method `virtual void updateMap(map<string int> &m)` to our book hierarchy. Or we could write a visitor.

```c++
class Catalogue : public BookVisitor {
  map<string, int> theCatalogue;
public:
  void visit(Book &b) override {
    ++theCatalogue[b.getAuthor()];
  }
  void visit(Text &t) override {
    ++theCatalogue[t.getTopic()];
  }
  void visit(Comic &c) override {
    ++theCatalogue[c.getHero()];
  }
};
```
`book.h` includes `BookVisitor.h` which includes `text.h`, which itself includes `book.h`. A circular dependancy. 

Because of `Book`'s include guard, when `Book` includes `BookVisitor` and therefore `Text`. `Text`'s inclusion of `Book` does not happen. So `Text` can't see that `Book` is a class. 

But are all of these includes really necessary? 

## Compilation Dependancy

When does a compilation dependancy exist. i.e. When does a file really need to include another. 

```c++
class A {
  ...
};

//b.h

class B : public A {
  ...
}; // Has a true compilation dependancy

//c.h

class C {
  A myA;
  ...
}; // Has a true compilation dependancy

//d.h

class D {
  A* myA;
  ...
} // Doesn't have a compilation dependancy

//e.h

class E {
  ...
  A f(A a);
} // Doesn't have a true compilation dependancy
```
Which of these need to include `a.h` and which only need a forward declaration?

classes `B` and `C` have compilation dependencies because in order for the compiler to know how large `B` and `C` are, it needs to know the size of `A`. 

Classes `D` and `E` do not. The compiler knows the size of a pointer and function prototypes are only used for typeChecking purposes. 

If there is no compilation dependancy, then do not include the file. Just a forward declaration. 

If `A` changes, `B` and `C` also need to be recompiled. But `D` and `E` don't.

Now in the implementation files, a true compilation dependancy is almost certain. 

```c++
#include "a.h"

void D::f() {
  myA->someFn();
}
```

Do the `#include` statements in your `.cc` files where possible. 

Now consider the `XWindow` class.

```c++
class XWindow {
  Display *d; // This is private data
  Window w; // Do we need to know what all of this means?
  int s; // Do we even care?
  GC gc;
  unsigned long Colours[10];
  ...
};
```

What if we add or change these private fields? Our client code must now be recompiled. How can we make it so that this isn't the case? 

**Solution**: is to use the Pointer to Implementation (pImpl) idiom. Create a second class `XWindowImpl` which stores the implementation details, `XWindow` just has a ptr to it. 

```c++
//XWindowImpl.h
#include <X11/XLib.h>
Struct XWindowImpl {
  Display *d;
  Window w;
  int s;
  GC gc;
  unsigned long Colours[10];
};
```
```c++
//window.h
class XWindowImpl;

class XWindow {
  XWindowImpl *pImpl;
public:
  //no change
};
```
```c++
//window.cc
#include "window.h"
#include "XWindowImpl.h"

XWindow::XWindow(...) : pImpl{new pImpl{...}} {...}
```
In other methods replace fields `w`, `d`, `s` etc with `pImpl->w`, `pImpl->d` etc. 

If you confine all `XWindow` private fields within `XWindowImpl`, then only `window.cc` needs to be recompiled if you change `XWindow`'s implementation. 

Generalization: What if there's multiple types of windows? e.g. `XWindow` and `YWindow`? Then make the `Impl Struct` a superclass. 

![pImple](res/pImple.png)

The pImple with a hierarchical Impl structures is called the *Bridge Pattern*.

## Measures of Design Quality

### Coupling and Cohesion

**Coupling**: The degree to which distinct program modules depend on each other.

**Low**
* Modules communicate through function calls with basic params/results.
* Modules pass arrays/structs back and forth
* Modules affect each other's control flow
* Modules share global data
* Modules have access to each other's implementations (friends)

**High**

High Coupling:
* Changes to one module require greater changes to other modules.
* Harder to reuse individual modules.

---
**Cohesion**: How closely related elements of a specific module are.

**Low**

* Arbitrary grouping of unrelated element (e.g. `<utility>`)
* Elements share a common theme, may be some base code, but otherwise unrelated (e.g. `<algorithms>`)
* Elements manipulate the state over a lifetime of an object (e.g. `open/read/closed`)
* Elements pass data to each other
* Elements cooperate to perform exactly one task

**High**

Low Cohesion:
* Poorly organized code.
* Harder to maintain, reuse.

**Goal: Low Coupling, High Cohesion**

## Decouple the interface (MVC) 

Your primary program classes should not be printing/displaying things. 

Example:
```c++
class ChessBoard {
  ...
  cout << "Your Move" << ...
};
```
This is bad design as it inhibits code reuse. 

What if you want to reuse the `ChessBoard` class, but not have it communicate with `stdout`? 

A better solution, give the class `Stream` objects with which it can do input/output.

```c++
class ChessBoard {
  istream &in;
  ostream &out;
public:
  ChessBoard (istream &in, ostream &o) ... 
  ... out << "Your Move" ...
};
```
This is better, but what if we don't want to use streams at all? e.g. graphics. 

Your `ChessBoard` should not be doing any communication at all.

**Single Responsibility Principle**: A class should have one reason to change. Game state and communication are two reasons.




