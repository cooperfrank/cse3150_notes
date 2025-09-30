# 09-25-2025

## Classes
- A class is a type.
- It describes a set of objects that:
    - Share some properties.
    - Meet some requirements.
- Adding properties or requirements:
    - Constrains the type.
    - Fewer objects have the properties or requirements.
        - Example: There are many students, but fewer **CSE3150** students specifically.
        - Called **subclasses**.

### Student Class Example
```cpp
#pragma once
#include <string>
#include <vector>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;
};
```

### Style Guidelines
- Classes (and types in general) start with uppercase.
- Methods (and functions in general) start with lowercase.
- Underscores:
    - Generally, don't use **leading underscores** (reserved in C++).
    - Use **trailing underscores** to indicate private attributes.

## Objects
- Objects are **instances of a class**.
    - Members of the set of objects denoted by the class.
- When a program starts:
    - It has a lot of classes.
    - Often, no instances yet.
        - You can declare classes at the start of runtime with `static`, but generally frowned upon.
    - Instances are generally created at runtime.

## Smart Pointers
- Raw pointers (`Student* s = new Student();`) require manual `delete`.
- Smart pointers automatically manage memory and help avoid leaks.
- Three types of smart pointers.

### `std::unique_ptr`
- Represents sole ownership of a resource.
- Cannot be copied, only moved.
- Automatically deletes the resource when it goes out of scope.

```cpp
#include <memory>

auto up = std::make_unique<Student>();
// auto up2 = up; // ERROR: cannot copy
auto up2 = std::move(up); // Ownership transferred
```

### `std::shared_ptr`
- Shared ownership: multiple pointers can manage the same object.
- Internally uses reference counting.
- Deletes the object **only** when the last owner goes out of scope.
- Very similar to Python's memory management (minus garbage collection).

```cpp
#include <memory>

auto sp1 = std::make_shared<Student>();
auto sp2 = sp1; // Both share ownership
// Resource destroyed only when both sp1 and sp2 go out of scope
```

#### Cycles with `shared_ptr`
- Two objects that reference each other with `shared_ptr` can cause a memory leak.
- Reference count never goes to zero.
- In Python, this is still garbage collected, but slowly.

### `std::unique_ptr` and `std::shared_ptr` Comparison
| Feature       | `unique_ptr`      | `shared_ptr`                   |
|||--|
| Ownership     | Single            | Multiple                       |
| Copyable?     | No (only move)    | Yes                            |
| Overhead      | None              | Reference counting (minimal)   |
| Use Case      | Strict Ownership  | Shared Resources               |

### Best Practices
- Prefer `std::make_unique` or `std::make_shared` over raw `new`.
- Default to `unique_ptr` unless shared ownership is truly needed.

### Other Smart Pointers
#### `std::weak_ptr<T>`
- Non-owning observer of a `shared_ptr`.
- Used to prevent cyclic references.

##### Fixing cycles with `weak_ptr`
- Two objects that reference each other with `shared_ptr` can cause a memory leak.
- Solution: Use `std::weak_ptr` for one of the links.
- Breaks the cycle because it does not increment the reference count.
- In Python, similar functionality comes from `weakref.proxy`.

### Student Class Example Continued
`student.h`
```cpp
#pragma once
#include <string>
#include <vector>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;
};
```

`student.cpp`
```cpp
#include "student.h"
#include <memory>
#include <iostream>

using std::cout, std::endl;

int main() {
    Student s; // Stored on stack

    // Modifying attributes of Student instance stored on stack
    s.name_ = "John";
    s.mt_ = 80;
    s.final_ = 90;
    s.homeworks_.push_back(65);
    s.homeworks_.push_back(95);

    std::shared_ptr<Student> sp = std::make_shared<Student>(); // Stored on heap; or use auto keyword
    // Could also be unique_ptr; can copy into a unique_ptr but not copy a unique_ptr to another var

    *sp = s;
    cout << "s.name_ = " << s.name_ << endl;
    cout << "sp->name_ = " << sp->name_ << endl; // Remember arrow syntax for pointer attribute from C
    // Name will be the same

    cout << "&s.name_ = " << &s.name_ << endl;
    cout << "&s.name_ = " << s->name_ << endl;
    // Copying name from stack onto heap, will have different addresses

    s.name_ = "Bert";
    cout << "&s.name_ = " << &s.name_ << endl;
    cout << "&s.name_ = " << s->name_ << endl;
    // Will have same addresses as above respectively after name_ change
}
```

## Going OO
- We must combine both state and behavior and have strong cohesion between both
- Must couple state and behavior
    - We must create abstractions for the Student classes
- Separation between interface and implementation
    - Like with a queue ADT, it can be implemented under the hood with a list or singly linked list or doubly linked list and the users of the class don't care they can still enqueue and dequeue like normal

`student.h`
```cpp
#pragma once
#include <string>
#include <vector>
#include <istream>
#include <ostream>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Don't need to define functions in class, can just define stubs like below and define the implementation of the functions elsewhere
    void read(std::istream& is);
    void print(std::ostream& os);
};
```

`student.cpp`
```cpp
#include "student.h"
#include <memory>
#include <iostream>

using std::cout, std::endl;

void Student::read(std::istream& is) { // Implementation of function stub defined here
    is >> name_ >> mt_ >> final_; // Note: Don't need to use self. or this. to access attributes
    int num_homeworks = 0;
    is >> num_homeworks;

    for (int i = 0; i < num_homeworks; i++) {
        int v;
        is >> v;
        homeworks_.push_back(v);
    }
}

void Student::print(std::ostream& os) {
    os << "Name: " << name_ << "[" << mt_ << "," << final_ << "]" << endl;

    for(int v : homeworks_) {
        os << "\t" << v << endl;
    }
}

int main() {
    Student s; // Stored on stack

    // Modifying attributes of Student instance stored on stack
    s.name_ = "John";
    s.mt_ = 80;
    s.final_ = 90;
    s.homeworks_.push_back(65);
    s.homeworks_.push_back(95);

    std::shared_ptr<Student> sp = std::make_shared<Student>(); // Stored on heap; or use auto keyword
    // Could also be unique_ptr; can copy into a unique_ptr but not copy a unique_ptr to another var

    *sp = s;

    return 0;
}
```

## Method Overloading
- C++ supports overloaidng methods, which is several definitions with:
    - Same name
    - Different number of arguments
    - Different types of arguments
- C++ disambiguates the call and selects the right method
- NOT the same as method overriding (which is related to inheritance)

## Class Lifetime
### Birth: Constructor
- Idea:
    - Go a memory pool (heap or stack) and grab memory for class
        - Either grab memory from stack (atomatic)
        - Or use keyword `new` to grab memory from heap
    - Then initialize
    - Execute a method when creating an object
    - Can oberload constructors (multiple ways to initialize)
    - Multiple kinds of constructors
        - Default, Customer, Copy Move

#### Default Constructor
`student.h`
```cpp
#pragma once
#include <string>
#include <vector>
#include <istream>
#include <ostream>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Defualt constructor; remember to define all variables that are not objects with their own default constructors
    Student() {
        mt_ = final_ = 0;
    }
    // std::string and std::vector have their own default constructors, we do not need to do it here

    // Constructor overloading
    Student(const std::string& s) {
        mt_ = final_ = 0;
        name_ = s;
    }

    // Initializer list; allows for increased performance
    Student(const std::string& s): name_(s), mt_(0), final_(0) {}

    void read(std::istream& is);
    void print(std::ostream& os);
};
```

#### Copy Constructor
- If you omit the copy constructor the compiler provides one for you
    - The compiler provided Copy Constructor is a Shallow Copy
        - Shallow Copy: Just makes pointers to the same place, not the individual values (like Python). This means that if the value is changed in one place, it also gets changed in the other.
- If you want to deep copy, you must implement it yourself, the compiler does not do it for you.

`student.h`
```cpp
#pragma once
#include <string>
#include <vector>
#include <istream>
#include <ostream>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Copy Constructor: Takes in a second student, and builds a new student
    Student(const Student& s2) {
        name_ = s2.name_;
        mt_ = s2.mt_;
        final_ = s2.final_;

        for (double d : s2.homeworks_) {
            homeworks_.push_back(d);
        }
        // Could also use std::vector's copy constructor: homeworks_(s2.homeworks_);
    }

    void read(std::istream& is);
    void print(std::ostream& os);
};
```

#### Move Constructor
- Purpose: Transfer ownership of resources instead of copying
    - Avoid expensive deep copies
    - Leaves TODO: FINISH

##### When to Write a Move Constructor
- If your class manages resourecs directly
- For classes that contain only STL containers
- Rule of 5: if you write one of the following, you often need to write the rest
    - Destructor
    - Copy constructor
    - Copy assignment operator
    - Move constructor
    - Move assignment operator

`student.h`
```cpp
#pragma once
#include <string>
#include <vector>
#include <istream>
#include <ostream>

struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Move Constructor
    TODO: FINISH

    void read(std::istream& is);
    void print(std::ostream& os);
};
```

## Lvalue
- Think like a house with an address, you can return to it anytime
- TODO: FINISH


## Rvalues
- Temporary object / literal
- No persistent address, vanishes after the stateemnt
- Cannot take its address directly
- Think like a hotel guest: exists briefly, then disappears

## T& vs T&&
- T& = reference to Lvalue
- T&& = reference to Rvalue
- TODO: FINISH

09-30-2025

## Transfer Operators
- Simple Idea:
    - Execute a method by invoking the assignment operator (ex. x = y; int z = 5;)
- When assigning an object, it's a little more complicated
    - You have the ability to redefine what happens when you transfer objects

```cpp
struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Transfer operator
    Student& operator=(const Student& s) {
        name_ = s.name_;
        mt_ = s.mt_;
        final_ = s.final_;
        homeworks_ = s.homeworks_;
        return *this; // Returns a reference to the object, useful for int x = y = z = Student()
    }
}
```

### Assignment Operator Niches
- Assignment operator takes a const reference:
    - Reference: For performance reasons, no copy
    - Const: When we assign something, we don't want to modify it!
- We return *this (a pointer to our student instance)

### `x = x`
- If we do `x = x`, every member of Student is copied to itself
    - Breaks for strings
- To prevent running all code for `x = x`, we can have the following line guard at the top of :
```cpp 
    // Transfer operator
    Student& operator=(const Student& s) {
        if (this == &s) {return *this} // Guard
        name_ = s.name_;
        mt_ = s.mt_;
        final_ = s.final_;
        homeworks_ = s.homeworks_;
        return *this; 
    }
```

### Overriding move operator

```cpp
struct Student {
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

    // Transfer operator
    Student& operator=(const Student&& s) {
        FINISH
    }
}
```

## Destructors
- Use destructors whenever we want to release memory held by the instance
    - ex. Memory, files, network connections (sockets)
- Easier to use shared pointer
    - No need to manually delete or reclaim
- Similar to dunder del in Python; invoked when the object dies
- No overloading destructor, only one per class
- Always starts with tilds
- Called implicitly
- Recursive in nature (ex. Student will call destructor on each member like string and vector)
- Often implicitly called when exiting the stack, or when delete is called to free the memory

```cpp
~Student() {
    std::cout << "Destroy!" << this << std::endl;
}
```

## Sidenote: Rust
- Copy constructors can be fairly complicated
- It's very important in general to stick to smart pointers and STL classes
- Rust, a type safe version of C++, actually fixes this by removing the copy constructors from C. Only moves are allowed by default (much safer)
- Also backwards compatible with C++ (you can write C++ code in Rust)

## Class Protection
- We wish to hide valuable bits and protect from intruders
- C style struct exposes everything by default, but we can change
- Three strategies:
    1. Public
    2. Protected
    3. Private

### Private
- Denoted using the `private:` header in the class header
- Note: all public methods that use the private variables must be implemented in the .cpp files since they haven't been defined in the scope
- Alternatively, you can define private variables first and then define the methods in the headers like normal since they have been defined
- Can also make functions like the assignment operator private

*Private Attributes Example*
```cpp
struct Student {
    // ...

private:
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;
}
```

*Private Attributes Example 2*
```cpp
struct Student {
private:
    std::string name_;
    double mt_, final_;
    std::vector<double> homeworks_;

public:
    // ...
}
```

### Public
- Default for **struct** attributes with no keyword
    - For backwards compatibility with C
- By default, everything in **classes** are private
- NOTE: The ONLY difference between structs and classes is that they are bt default public and private respectively
    - Generally, structs are used to keep track of data without behavior
    - Classes are used to keep track of data and behavior
    - This is just the convention

## Contract
- C++ supports the separation of:
    - Contract (the what)
    - Implementation (the how)
- Two mechanisms
    - Header and implementation file
    - Abstract classes (yet to cover)


