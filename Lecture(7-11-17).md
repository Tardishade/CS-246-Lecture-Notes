## Error Handling (7/11/17)
`v[i]` gets the ith element of `v` is unchecked
`v.at(i)` gets the ith element of `v`, is checked, but what should it do if an invalid index is accessed?

Error handling by definition is a non-local problem. Vector can detect the error, but doesn't know how the client wants to respong. Client knows what they want to do if an error occurs but can't detect it. 

* C-Style solution: Have vector  methods return a special err status code as their return value or set a global error variable. These are bad solutions

* C++ Solution: When an error condition arises the function raises an *exception*

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

Refer to [Diag 1]()

The abstract class subject has all the common code to all subjects. The abstract class observer holds only the interface common to all observers. 

### Sequence of Method Calls
1. Subject's state is updated
2. `Subject::notifyObservers()` calls each observer's `notify` method.
3. Each observer calls `ConcreteSubject::getState to get the data and reacts to it. 

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

class Better : public Observer {
    Horserace * subject;
    String name, myHorse;
    public:
        Better(...) :... {
            subject->attach(this);
        }
        void notify() {
            String winner = subject->getState();
            cout<<(winner == myHorse? "win" : "lose")<<endl;
        }
};
