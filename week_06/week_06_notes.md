# Week 6 Notes
Cooper Frank | CSE 3150 | Prof. Justin Furuness

10-02-2025, 10-07-2025

## Four Pillars to OOP
### Encapsulation
- "has-a" relationship
    - ex. a car *has a* wheel
- Purpose is to bind things together as a cohesive package
- `struct` and `class` are used to offer this feature 
    - Remember: both have the same mechanics, but the only difference is the default for privacy
        - `struct`: by default, everything is *public*
        - `class`: by default, everything is *private*

*Example: `Point` class definition and implementation*
`point.h`
```cpp
#pragma once
#include <ostream>

class Point {
    double x_;
    double y_;

public:
    Point() {x_ = y_ = 0;} // Default constructor
    Point(double x, double y) {x_ = x; y_ = y;} // Custom constructor
    Point(const Point& p) : x_(p.getX()), y_(p.getY()) {} // Copy constructor (with initializer list notation)

    double getX() const {return x_;} // const indicates will not modify instance
    double getY() const {return y_;}
    void set(double x, double y) {x_ = x; y_ = y;}

    Point& operator=(const Point& p) {
        x_ = p.getX();
        y_ = p.getY();
        return *this;
    }

    friend Point operator+(const Point& p1, const Point& p2);
    friend Point operator*(const Point& p1, const double s);
    friend std:ostream& operator<<(std::ostream& os, const Point& p);

    // Remember: `friend` are general functions as opposed to call on this
    // friend gives access to private variables
};
```

`point.cpp`
```cpp
#include "point.h"
#include <ostream>

Point operator+(const Point& p1. const Point& p2) {
    return Point(p1.getX() + p2.getX(), p1.getY() + p2.getY());
}

Point operator*(const Point& p, double s) {
    return Point(p.getX() * s, p.getY() * s);
}

std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << "(" << p.getX() << ", " << p.getY() << ")";
    return os;
}
```

`rectangle.h`
```cpp
#pragma once
#include <memory>
#include <ostream>
#include "point.h"

class Rectangle {
    std::shared_ptr<Point> corner_;
    double w_;
    double h_;

public:
    Rectangle() : corner_(new Point), w_(0), h_(0) {}
    Rectangle(double x, double y, double w, double h) : corner_(new Point(x, y)), w_(w), h_(h) {}

    std::shared_ptr<Point> getCorner() const {return corner_;}
    double getWidth const {return w_;}
    double getHeight const {return h_;}

    friend std::ostream& operator<<(std::ostream& os, const Rectangle& r) {
        os << "Rectangle(" << *r.corner_ << ", " << r.w_ << ", " << r.h_ << ")";
        return os;
    }
};
```

`main.cpp`
```cpp
#include "point.h"
#include "rectangle.h"
#include <iostream>

using std::cout, std::endl;

int main() {
    // Point testing
    Point a(1.0, 2.0);
    Point b(3.0, 4.0);
    Point c = a + b;
    Point d = c * 2.0;

    cout << "a = " << a << endl;
    cout << "b = " << b << endl;
    cout << "c = " << c << endl;
    cout << "d = " << d << endl;

    // Rectangle testing
    Rectangle* r = new Rectangle(10, 20, 100, 100);
    cout << "Before: " << *r << endl;
    Point p1(42, 42);
    *(r->getCorner()) = p1;
    cout << "After: " << *r << endl;
    delete r;

    return 0;
}
```

#### Example Analysis: Rectangle Object Model
- Stack has `p`
- `p` points to a Rectangle object on the Heap
- `corner_` points to a shared_ptr
- The shared_ptr points to a `Point` object on the Heap

TODO: FINISH

#### Problem with this model
- We can move our rectangles
- This does not make sense, rectangles shouldn't be moving around
- Encapsulation is easy to break
- Implementation should be separate from contract
- We should not let a user manipulate the "guts" of the rectangle, just like how we don't let users modify the middle of a stack
- Imagine that there is a giant box around our object, nothing should get in

#### The Fix
- Simply allocate the point *in the class* on the stack!
- Now, if getCorner is called, it returns a copy of the point, rather than an object on the heap
- This is a much cleaner solution! Everything is colated in memory (right next to each other)

*Fixed `rectangle.h`*
`rectangle.h`
```cpp
class Rectangle {
    Point corner_; // Stack allocated
    double w_;
    double h_;

public:
    Rectangle() : w_(0), h_(0) {}
    Rectangle(double x, double y, double w, double h) : corner_(x, y), w_(w), h_(h) {} // Constructor for Point corner_ 

    std::shared_ptr<Point> getCorner() const {return corner_;}
    double getWidth const {return w_;}
    double getHeight const {return h_;}

    friend std::ostream& operator<<(std::ostream& os, const Rectangle& r) {
        os << "Rectangle(" << *r.corner_ << ", " << r.w_ << ", " << r.h_ << ")";
        return os;
    }
};
```

### Inheritance
- Promotes:
    1. Code reuse
    2. Code specialization
- Related concepts
    1. Subtyping
    2. Subclasses
    3. Abstract classes
    4. etc.
- Purpose
    - Pragmatically: code reuse
    - In Theory: Type refinements (subtyping through subclasses)

#### Hierarchy
- With each specialization
    - Requirements grow
    - The set of entities that meet those requirements shrink
- Specialization is akin to
    - Making subsets
    - Making subtypes

#### Requirements
- Can take different forms
- Mainly:
    1. Attributes
    2. Specific behaviors
- ex. For `Car`: Attribute of four wheels, behavior of electric charging or gas

#### Subtyping
- Let A, B be two types
    - We write A <: B to state that A is a subtype of B
- ex. A = Car, B = Vehicle, A <: B 

#### Subsumption
- Whenever A <: B, anytime a B is expected, one can provide an A
    - In other words, A is a subtype/subset of B, therefore **every instance of A is also an instance of B**

*Vehicles and Cars Code Example*
`inheritance/vehicles.h`
```cpp
#pragma once
#include <iostream>

class Vehicle {
public:
    Vehicle() {}
    void print() {
        cout << "I (" << this << ") am a vehicle!" << endl;
    }
}

class Car: public Vehicle { // Inheritance
public:
    Car() {}
    void print() {
        cout << "I (" << this << ") am a car!" << endl;
    }
}
```

`inheritance/main.cpp`
```cpp
#include "vehicles.h"

void foo(Vehicle& v) {
    v.print();
}

int main() {
    Vehicle v;
    Car c;

    v.print();
    c.print();

    foo(v);
    foo(c);

    return 0;
}
```

##### Example Analysis: Subsumption
- Here `Car` is inheriting from `Vehicle`; this is the syntax
- Both calls to foo are fine so long as subsumption is present
    - Even though foo expects a vehicle, we can pass a car
    - Object identity (address) is preserved
- Nothing under the hood is being changed about the car

So why do we get "I'm a vehicle" from foo(car)?

- This is called polymorphism
    - In C++, you can define your classes without polymorphism
        - This is what we've done thus far
        - Polymorphism must be explicitly defined
    - As we defined our classes, when we pass car into functions for vehicle, it will call the vehicle implementation methods

#### Inheritance and Privacy
- When inheriting you can alter the privacy of what you inherit
    - Public
    - Protected
    - Private
- For example, if UG is a superclass of Grad and Grad inherits privately, then only Grad can call UGs methods, not the outside world

##### Public Inheritance
- What you inherit keeps its pervious states
    - Public remains public
    -
- Whatever you override uses the privacy state of the inherited
- The common case
- When A publicly inherits from B, 

TODO: FINISH

##### Protected Inheritance
- What you inherit CHANGES its previous status
    - Public becomes protected
    -
- Not the default
    - `class A : protected B`
- When A protected inherits from B:
    - A and its sublcasses know that they inherit from B
    - Nobody else knows
    - Subsumption does not work

##### Private Inheritance
- What you inherit changes its previous status
    - Everything becomes private (public, protected, private)
    - Whatever you override uses the privacy setting of the function
- The default for classes
- When A private inherits from B
    - A knows it inherits from B
    - A's subclasses do not know
    - Everything else does not know
    - Inheritance fact is completely hidden
- Often used when you are in the mindset of "code reuse"
    - Does not really stick with the nice theoretical overview of OO
    - But C++ supports both

#### Inheritance vs Overriding
- When A inherits from B, it:
    - Has as all attributes of B
    - Has all methods of B
    - Can add attributes or methods
    - Upgrade or refine some methods
        - This is overriding, NOT overloading

*Inheritance example: Grading Policy*
`grading_policy.cpp`
```cpp
#include <ostream>
#include <string>
#include <iostream>

class Undergraduate {
protected:
    std::string name_;
    double grade_;

public:
    Undergraduate(const std::string& n, double g): name_(n), grade_(g) {}
    const char letterGrade() const;
    friend std::ostream& operator<<(std::ostream& os, const Undergraduate& ug) {
        return os << "UG = (" << ug.name_ << ", " << ug.grade_ << ", " << ug.letterGrade() << ")";
    }
};

class Graduate: public Undergraduate {
// Has access to name_ and grade_ variables from inheritance
public:
    Graduate(const std::string& n, double g): Undergraduate(n, g) {}
    const char letterGrade() const;
    friend std::ostream& operator<<(std::ostream& os, const Graduate& g) {
        return os << "G = (" << g.name_ << ", " << g.grade_ << ", " << g.letterGrade() << ")";
    }
};

const char Undergraduate::letterGrade() const {
    if (grade_ > 80 && grade_ <= 100) {
        return 'A';
    }
    else {
        return 'F';
    }
}

const char Graduate::letterGrade() const {
    if (grade_ > 95 && grade_ <= 100) {
        return 'A';
    }
    else {
        return 'F';
    }
}

int main() {
    Undergraduate s1("Bernard", 61);
    Graduate s2("Bob", 81);
    Undergraduate& gr = s2;

    std::cout << s1 << std::endl; // UG = (Bernard, 61, F)
    std::cout << s2 << std::endl; // G = (Bob, 81, F)
    std::cout << gr << std::endl; // UG = (Bob, 81, A)
}
```

- We see again that polymorphism is NOT the default in C++
    - Polymorphism is expensive

#### Dynamic Binding (`virtual`)
- We have inheritance and lack polymorphism. The solution is dynamic binding
- We have compile time polymorphism
    - Method overloading
    - Template functions and classes
- Now, we talk about RUNTIME polymorphism
    - This is dynamic binding of overridden (NOT overloading) methods
- Purpose: Provide the ability to respond to messages based on the dynamic type rather than the compile time type
    - In Java and Python, this is the default
- `virtual` keyword switches the method from static binding (compile-time types) to dynamic binding (runtime types)

```cpp
class Undergraduate {
protected:
    std::string name_;
    double grade_;

public:
    Undergraduate(const std::string& n, double g): name_(n), grade_(g) {}
    virtual const char letterGrade() const; // Added virtual keyword; all child classes will have this as virtual method
    virtual std::string kind() const { // Added virtual keyword; all child classes will have this as virtual method
        return "Undergraduate";
    }
    friend std::ostream& operator<<(std::ostream& os, const Undergraduate& ug) {
        return os << ug.kind() << " = (" << ug.name_ << ", " << ug.grade_ << ", " << ug.letterGrade() << ")";
    }
};

class Graduate: public Undergraduate {
// Has access to name_ and grade_ variables from inheritance
public:
    Graduate(const std::string& n, double g): Undergraduate(n, g) {}
    virtual const char letterGrade() const; // Could also remove virtual keyword since parent class has it defined as virtual
    virtual std::string kind() const { // Could also remove virtual keyword since parent class has it defined as virtual
        return "Graduate";
    }
    // We use inheritance for operator<<
};
```

#### Tricky Inheritance
- Space usage and memory layout
- Overload and overrides
- Deallocation
- Multiple Inheritance
- Diamond Inheritance

##### Space and Memory
An object contains:

- (Optionally) A VPTR, which is a pointer to a table containing pointers to dynamically bound methods
- A collection of fields for attributes

```cpp
class X {
    int a_;
    double b_;
    std::string c_;
public:
    X();
    virtual std::string getName();
    int getA();
};
```

###### 1. Class `X` Memory Layout (With Dynamic Binding (`virtual`))

| **Memory Layout** | **Description**                                          |
| ----------------- | -------------------------------------------------------- |
| **VPTR**          | Pointer to VTABLE for class `X` (dynamic dispatch table) |
| **a_**            | `int` field of object                                    |
| **b_**            | `double` field of object                                 |
| **c_**            | `std::string` field of object                            |

**VTABLE for `X`:**

| **VTABLE**  | **Points to**             |
| ----------- | ------------------------- |
| `getName()` | Address of `X::getName()` |

###### 2. Class `X` Memory Layout (Static Binding (no `virtual`))

| **Memory Layout** | **Description**               |
| ----------------- | ----------------------------- |
| **a_**            | `int` field of object         |
| **b_**            | `double` field of object      |
| **c_**            | `std::string` field of object |

**Method calls:** Resolved *statically* at compile-time; no table involved.

###### Summary of Differences

| Feature                    | Virtual (Dynamic) | Non-Virtual (Static) |
| -------------------------- | ----------------- | -------------------- |
| Table pointer (VPTR)       | Yes               | No                   |
| Method table (VTABLE)      | Yes               | No                   |
| Method resolution          | Runtime           | Compile-time         |
| Memory overhead per object | +size of pointer, size of lookup table  | None                 |


##### Overloading vs. Overriding
```cpp
#include <iostream>

class B {
public:
    virtual void f(short a) {
        std::cout << "In B" << std::endl;
    }
};

class C: public B {
    virtual void f(int a) { // (!) Overloaded, not overwritten; parameters are different
        std::cout << "In C" << std::endl;
    }
    // Overwritten with keyword override, will raise compiler error if it's not overriding to catch overloading mistakes
    // Keyword final denotes that this function cannot be overridden by a child class
    virtual void f(short a) override final { 
        std::cout << "In C" << std::endl;
    }
};

class D: public C {
    void f(int) override { // Will throw error, cannot override final function from C
        std::cout << "In D" << std::endl;
    }
}

int main() {
    B* aPtr = new C;
    aPtr->f(1); // Prints "In C"
    delete aPtr;
    return 0;
}
```

##### Deallocation
Consider an example where class D inherits from class C which inherits from class B which inherits from class A:

- When you call delete on a pointer to A and the actual object is of type D, C++ ensures all destructors run correctly
- If A's destrictor is virtual, then
    - The destructor of D is invoked first
    - After D frees its members, C's destructor runs
    - Then B's destructor
    - Finally, A's destructor
- Each class only cleans up its own members, the chain ensures the entire object is safely destroyed
- Without a virtual destructor in A, only A's destructor runs, which leads to resource leaks

`grading_policy.cpp`
```cpp
#include <ostream>
#include <string>
#include <iostream>

class Undergraduate {
protected:
    std::string name_;
    double grade_;

public:
    Undergraduate(const std::string& n, double g): name_(n), grade_(g) {}
    virtual ~Undergraduate() {}; // Allows destructor chain
    const char letterGrade() const;
};

class Graduate: public Undergraduate {
public:
    Graduate(const std::string& n, double g): Undergraduate(n, g) {}
    const char letterGrade() const;
    
};

const char Undergraduate::letterGrade() const {
    // ...
}

const char Graduate::letterGrade() const {
    // ...
}

int main() {
    Undergraduate s1("Bernard", 61);
    Undergraduate* s2 = new Grad("Billy", 67)

    std::cout << s1 << std::endl; 
    std::cout << s2 << std::endl; 

    delete s2;
    return 0;
}
```