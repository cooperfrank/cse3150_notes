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

