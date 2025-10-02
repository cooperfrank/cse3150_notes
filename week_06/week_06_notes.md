# Four Pillars to OOP
## Encapsulation
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

### Example Analysis: Rectangle Object Model
- Stack has `p`
- `p` points to a Rectangle object on the Heap
- `corner_` points to a shared_ptr
- The shared_ptr points to a `Point` object on the Heap

TODO: FINISH

### Problem with this model
- We can move our rectangles
- This does not make sense, rectangles shouldn't be moving around
- Encapsulation is easy to break
- Implementation should be separate from contract
- We should not let a user manipulate the "guts" of the rectangle, just like how we don't let users modify the middle of a stack
- Imagine that there is a giant box around our object, nothing should get in

### The Fix
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

## Inheritance
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

### Hierarchy
- With each specialization
    - Requirements grow
    - The set of entities that meet those requirements shrink
- Specialization is akin to
    - Making subsets
    - Making subtypes

### Requirements
- Can take different forms
- Mainly:
    1. Attributes
    2. Specific behaviors
- ex. For `Car`: Attribute of four wheels, behavior of electric charging or gas

### Subtyping
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

#### Example Analysis: Subsumption
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

### Inheritance and Privacy
- When inheriting you can alter the privacy of what you inherit
    - Public
    - Protected
    - Private
- For example, if UG is a superclass of Grad and Grad inherits privately, then only Grad can call UGs methods, not the outside world

#### Public Inheritance
- What you inherit keeps its pervious states
    - Public remains public
    -
- Whatever you override uses the privacy state of the inherited
- The common case
- When A publicly inherits from B, 

TODO: FINISH

#### Protected Inheritance
- What you inherit CHANGES its previous status
    - Public becomes protected
    -
- Not the default
    - `class A : protected B`
- When A protected inherits from B:
    - A and its sublcasses know that they inherit from B
    - Nobody else knows
    - Subsumption does not work

#### Private Inheritance
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

### Inheritance vs Overriding
- When A inherits from B, it:
    - Has as all attributes of B
    - Has all methods of B
    - Can add attributes or methods
    - Upgrade or refine some methods
        - This is overriding, NOT overloading
