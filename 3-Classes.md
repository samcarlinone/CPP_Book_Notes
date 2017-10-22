# Classes
_(The C++ Programming Language Chapter 3, ed. 4, by Bjarne Stroustrup)_

### Concrete Types

##### Demo complex number class
```c++
class complex {
  double re, im; // representation: two doubles
public:
  complex(double r, double i) :re{r}, im{i} {} // construct complex from two scalars
  complex(double r) :re{r}, im{0} {} // construct complex from one scalar
  complex() :re{0}, im{0} {} // default complex: {0,0}
  double real() const { return re; }
  void real(double d) { re=d; }
  double imag() const { return im; }
  void imag(double d) { im=d; }
  complex& operator+=(complex z) { re+=z.re , im+=z.im; return ∗this; } // add to re and im
  // and return the result
  complex& operator−=(complex z) { re−=z.re , im−=z.im; return ∗this; }
  complex& operator∗=(complex); // defined out-of-class somewhere
  complex& operator/=(complex); // defined out-of-class somewhere
};
```
Simple operations such as `+=` are __inlined__ meaning they make no function calls when compiled

Functions defined in class are inlined by default

`const` keyword on member functions indicate they don't modify the object

__Pass by value functions automatically copy objects__


---

##### Destructors
```c++
class Vector {
private:
  double∗ elem; // elem points to an array of sz doubles
  int sz;

public:
  Vector(int s) :elem{new double[s]}, sz{s} // constructor: acquire resources
  {
    for (int i=0; i!=s; ++i) elem[i]=0; // initialize elements
  }

  ˜Vector() { delete[] elem; } // destructor: release resources

  double& operator[](int i);
  int size() const;
};
```

`~` before class name is a destructor, `new` allocates memory from heap, `delete` frees heap memory

__*RAII*__ or _Resource Acquisition Is Initialization_ allows the elimination of "naked `new` operators"

By keeping the allocation / de-allocation within the object we help avoid memory leaks

##### Initializing

```c++
class Vector {
public:
  Vector(std::initializer_list<double>); // initialize with a list
  // ...
  void push_back(double); // add element at end increasing the size by one
  // ...
};
```

`initializer_list` is a type in the std, represents a curly brace list initializer

```c++
//Actual initializer_list code
Vector::Vector(std::initializ er_list<double> lst) // initialize with a list
  :elem{new double[lst.size()]}, sz{lst.size()}
{
  copy(lst.begin(),lst.end(),elem); // copy from lst into elem
}
```

### Abstract Types

```c++
class Container {
public:
  virtual double& operator[](int) = 0; // pure virtual function
  virtual int size() const = 0; // const member function
  virtual ˜Container() {} // destructor
};
```

`virtual` means will be redefined later, a non-virtual function could be overridden,
but if a collection with the same type as the base class was called then the base class version
of the method would be used, `virtual` functions will almost always use the "lowest" definition

__Virtual vs. non-virtual can be very important to construction / destruction, if incorrectly implemented the
base destructor may be used, this is a common source of memory leaks__

`=0` designates the function as pure virtual, requiring deriving classes to define it
pure virtual functions are used to make a class abstract

```c++
class Vector_container : public Container { // Vector_container implements Container
  Vector v;
public:
  Vector_container(int s) : v(s) { } // Vector of s elements
  ˜Vector_container() {}
  double& operator[](int i) { return v[i]; }
  int size() const { return v.size(); }
};
```

example of implementing class, `public` after class means "is subclass of"

`unique_ptr` can be used as an alternative to returning "naked pointers", as it will destroy
the object it references when it goes out of scope
```c++
unique_ptr<Shape> read_shape(istream& is) // read shape descriptions from input stream is
//...
```

# Copy and Move

By default, object members are copied, often not desired behavior
```c++
class Vector {
private:
  double∗ elem; // elem points to an array of sz doubles
  int sz;

public:
  Vector(int s); // constructor: establish invariant, acquire resources
  ˜Vector() { delete[] elem; } // destructor: release resources

  Vector(const Vector& a); // copy constructor
  Vector& operator=(const Vector& a); // copy assignment

  double& operator[](int i);
  const double& operator[](int i) const;

  int size() const;
};

Vector::Vector(const Vector& a) // copy constructor
  :elem{new double[sz]}, // allocate space for elements
  sz{a.sz}
{
  for (int i=0; i!=sz; ++i) // copy elements
    elem[i] = a.elem[i];
}
```

sometimes we want to move instead of copy
```c++
class Vector {
// ...
  Vector(const Vector& a); // copy constructor
  Vector& operator=(const Vector& a); // copy assignment

  Vector(Vector&& a); // move constructor
  Vector& operator=(Vector&& a); // move assignment
};

Vector::Vector(Vector&& a)
  :elem{a.elem}, // "grab the elements" from a
  sz{a.sz}
{
  a.elem = nullptr; // now a has no elements
  a.sz = 0;
}
```

`&&` is an "rvalue reference", in this case r and l value refer to which side of an argument a value
might appear on, in this case an "rvalue reference" cannot be assigned to

> A move operation is applied when an rvalue reference is used as an initializer or as the righthand side of an assignment.

##### Suppressing operations
we must prevent base class move and copy being called on child classes
```c++
class Shape {
public:
  Shape(const Shape&) =delete; // no copy operations
  Shape& operator=(const Shape&) =delete;

  Shape(Shape&&) =delete; // no move operations
  Shape& operator=(Shape&&) =delete;

  ˜Shape();
  // ...
};
```
if we explicitly define a destructor, a copy and move constructor should not be automatically added

`=delete` is a general mechanism that can be used to suppress any operation

# Templates
`template<typename T>` makes `T` a type that will be defined later
```c++
template<typename T>
class Vector {
private:
  T∗ elem; // elem points to an array of sz elements of type T
  int sz;
public:
  Vector(int s); // constructor: establish invariant, acquire resources
  ˜Vector() { delete[] elem; } // destructor: release resources

  // ... copy and move operations ...

  T& operator[](int i);
  const T& operator[](int i) const;
  int size() const { return sz; }
};

template<typename T>
Vector<T>::Vector(int s)
{
  if (s<0) throw Negative_size{};
  elem = new T[s];
  sz = s;
}

template<typename T>
const T& Vector<T>::operator[](int i) const
{
  if (i<0 || size()<=i)
    throw out_of_range{"Vector::operator[]"};
  return elem[i];
}
```

now we can define Vectors of any type
```c++
Vector<char> vc(200); // vector of 200 characters
Vector<string> vs(17); // vector of 17 strings
Vector<list<int>> vli(45); // vector of 45 lists of integers
```
as we see, it is possible to nest types

we can also use templates in a much more complex way
```c++
template<typename Container, typename Value>
Value sum(const Container& c, Value v)
{
  for (auto x : c)
    v+=x;
  return v;
}

void user(Vector<int>& vi, std::list<double>& ld, std::vector<complex<double>>& vc)
{
  int x = sum(vi,0); // the sum of a vector of ints (add ints)
  double d = sum(vi,0.0); // the sum of a vector of ints (add doubles)
  double dd = sum(ld,0.0); // the sum of a list of doubles
  auto z = sum(vc,complex<double>{}); // the sum of a vector of complex<double>
  // the initial value is {0.0,0.0}
}
```

### Function Objects

```c++
template<typename T>
class Less_than {
  const T val; // value to compare against
public:
  Less_than(const T& v) :val(v) { }
  bool operator()(const T& x) const { return x<val; } // call operator
};

Less_than<int> lti {42}; // lti(i) will compare i to 42 using < (i<42)
Less_than<string> lts {"Backus"}; // lts(s) will compare s to "Backus" using < (s<"Backus")

void fct(int n, const string & s)
{
  bool b1 = lti(n); // true if n<42
  bool b2 = lts(s); // true if s<"Backus"
  // ...
}
```

the `operator()` allows the object to be "called", which is what makes this work

A more advanced example

```c++
template<typename C, typename P>
int count(const C& c, P pred)
{
  int cnt = 0;
  for (const auto& x : c)
    if (pred(x))
      ++cnt;
  return cnt;
}

void f(const Vector<int>& vec, const list<string>& lst, int x, const string& s)
{
  cout << "number of values less than " << x
    << ": " << count(vec,Less_than<int>{x})
    << '\n';
  cout << "number of values less than " << s
    << ": " << count(lst,Less_than<string>{s})
    << '\n';
}
```

to avoid having to define the `Less_than` function object it can be created implicitly

```c++
void f(const Vector<int>& vec, const list<string>& lst, int x, const string& s)
{
  cout << "number of values less than " << x
    << ": " << count(vec,[&](int a){ return a<x; })
    << '\n';
  cout << "number of values less than " << s
    << ": " << count(lst,[&](const string& a){ return a<s; })
    << '\n';
}
```

`[&](int a){ return a<x; }` is a _lambda expression_, the symbols inside the preceding `[]` are called the _capture list_

- `[&]` passes all local names will be passed by reference
  - `[&x]` this syntax will pass only x by reference
- `[=]` captures by value rather than reference, can be limited like `[&]`
- `[]` is also valid and will capture nothing

### Variadic Templates

these _variadic templates_ can accept an arbitrary number of arguments of arbitrary type

```c++
template<typename T, typename ... Tail>
void f(T head, Tail... tail)
{
  g(head); // do something to head
  f(tail...); // try again with tail
}

void f() { } // do nothing
```

here `...` (ellipsis) indicates the rest of the list,
thus we can see this function is called repeatedly on the shrinking list

also note that we have defined a separate function to handle when the list is empty

### Aliases

the `using` keyword can also be used to create type aliases, and new complex templates
```c++
template<typename C>
using Element_type = typename C::value_type;

template<typename Container>
void algo(Container& c)
{
  Vector<Element_type<Container>> vec; // keep results here
  // ...
}
```

or for a more complex example
```c++
template<typename Key, typename Value>
class Map {
  // ...
};

template<typename Value>
using String_map = Map<string,Value>;

String_map<int> m; // m is a Map<string,int>
```
