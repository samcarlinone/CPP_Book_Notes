# Classes
_(The C++ Programming Language Chapter 17, ed. 4, by Bjarne Stroustrup)_

### virtual Destructors
For classes with virtual functions, the destructor should also be declared virtual

### Initializing
```c++
class Person {
  string name;
  string address;
  // ...
  Person(const Person&);
  Person(const string& n, const string& a);
};

Person::Person(const string& n, const string& a)
  : name{n}
{
  address = a;
}
```

here name is initialized with a copy, address is first initialized empty then copied too

### In class initializers
we can use `{}` syntax to initialize members
```c++
class A {
public:
  A() {}
  A(int a_val) :a{a_val} {}
  A(D d) :b{g(d)} {}
  // ...
private:
  int a {7}; // the meaning of 7 for a is ...
  int b {5}; // the meaning of 5 for b is ...
  HashFunction algorithm {"MD5"}; // Cryptographic hash to be applied to all As
  string state {"Constructor run"}; // String indicating state in object lifecycle
};
```

### static Member Initialization
```c++
class Node {
  // ...
  static int node_count; // declaration
};

int Node::node_count = 0; // definition
```


### Copy
- Copy Constructor `X(const X&)`
- Copy assignment `X& operator=(const X&)`

#### Slicing
__Beware, a pointer of a child class can be implicitly converted to a pointer of the base class. This can cause data loss in copy operations__


### Move
```c++
Matrix(const Matrix&); // copy constr uctor
Matrix(Matrix&&); // move constr uctor
Matrix& operator=(const Matrix&); // copy assignment
Matrix& operator=(Matrix&&); // move assignment
```

### Explicit defaults
```c++
class Gslice {
public:
  Gslice() = default;
}
```

### Deleted functions
```c++
class Base {
  Base& operator=(const Base&) = delete;// disallow copying
  Base(const Base&) = delete;

  Base& operator=(Base&&) = delete; // disallow moving
  Base(Base&&) = delete;
};
```
