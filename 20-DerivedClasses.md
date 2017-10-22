# Derived Classes
_(The C++ Programming Language Chapter 20, ed. 4, by Bjarne Stroustrup)_

### Derived Classes
```c++
class Employee; // declaration only, no definition
class Manager : public Employee { // error : Employee not defined
// ...
};
```

public and protected members are available to derived classes

```c++
void Manager::print() const
{
  Employee::print(); // Must use ::
}
```

### Virtual Functions
Virtual functions allows us to call the function of a base class and have derived classes properly override

should avoid using virtual methods from within constructors or destructors

### Explicit Qualification
we can avoid the `virtual` system from within our objects thus

```c++
void Manager::print() const
{
  Employee::print(); //Not a virtual call
  // ...
}
```

### Override Control
- `virtual` may be overridden
- `=0` the function must be `virtual` and must be overridden
- `override` the function is meant to override a virtual function in a base class
- `final` the function is not meant to be overridden

#### override
```c++
struct D : B5 {
  void f(int) const override; // error : B0::f() is not virtual
  void g(int) override; // error : B0::f() takes a double argument
  virtual int h() override; // error : no function h() to override
};

//Not repeated in out of class definition
void Derived::f() override // error : override out of class
{
  // ...
}
;
```

`override` is a contextual keyword, meaning we can declare things with the name "override", don't do this

#### final
functions declared `virtual` can be overridden, but sometimes a child class may want to prevent further overriding

for this we use the `final` keyword

```c++
struct Node { // interface class
  virtual Type type() = 0;
  // ...
};

class If_statement : public Node {
public:
  Type type() override final; // prevent further overriding
  // ...
};
```

like `override`, `final` cannot be repeated out of class

also like `override`, it is a contextual keyword, don't abuse this plz

### using Base Members
functions do not overload across scopes

```c++
struct Base {
  void f(int);
};

struct Derived : Base {
  void f(double);
};

void use(Derived d)
{
  d.f(1); // call Derived::f(double)
  Base& br = d
  br.f(1); // call Base::f(int)
}
```

one way of addressing this problem is `using` and namespaces

```c++
struct D2 : Base {
  using Base::f; // bring all fs from Base into D2
  void f(double);
};

void use2(D2 d)
{
  d.f(1); // call D2::f(int), that is, Base::f(int)
  Base& br = d
  br.f(1); // call Base::f(int)
}
```

#### Inheriting Constructors
```c++
template<class T>
struct Vector : std::vector<T> {
  using vector<T>::vector; // inherit constructors
  T& operator=[](size_type i) { check(i); return this−>elem(i); }
  const T& operator=(size_type i) const { check(i); return this−>elem(i); }
  void check(size_type i) { if (this−>size()<i) throw Bad_index(i); }
};

Vector<int> v { 1, 2, 3, 5, 8 }; // OK: use initializer-list constructor from std::vector
```

### Return Type Relaxation
We can return `*D` or `D&` from a function that overrides a function that returns `*B` or `B&` provided `D` is a child class of `B`

## Abstract Classes
A class with one or more "pure-virtual" functions (`=0`) is abstract and cannot be instantiated

typically have a `virtual` destructor

## Access Control
- `private`: member functions and friends
- `protected`: derived class's member functions and friends
- `public`: anyone

### protected Members
suppose we wanted a public bound checking accessor for other classes, but a child class didn't want to lose efficiency

we could specify a second unchecked accessor, and declare it `protected` to stop external use

### Access to Base Classes
```c++
class X : public B {  };
class Y : protected B {  };
class Z : private B {  };
```
- `public`: `X` is a kind of `B`
- `private` and `protected`: `B` is an implementation detail of `Y` or `Z`

generally people expect base classes to be `public` so this specifier is commonly used with `class` but can be omitted for `struct`

### using-Declarations and Access Control
a `using`-declaration cannot be used gain access to additional information

however it can be used to reveal some members of a parent class

```c++
class BB : private B { // give access to B::b and B::c, but not B::a
public:
  using B::b;
  using B::c;
};
```

## Pointers to Members
`->*` and `.*` are infrequently used, but allow indirect access to object properties

### Pointers to Function Members
```c++
using Pstd_mem = void (Std_interface::∗)(); // pointer-to-member type

void f(Std_interface∗ p)
{
  Pstd_mem s = &Std_interface::suspend; // pointer to suspend()
  p−>suspend(); // direct call
  p−>∗s(); // call through pointer to member
}

//=====================[More Complex Example]
map<string,Std_interface∗> variable;
map<string,Pstd_mem> operation;

void call_member(string var, string oper)
{
  (variable[var]−>∗operation[oper])(); // var.oper()
}
```

### Pointers to Data Members
```c++
struct C {
  const char∗ val;
  int i;

  void print(int x) { cout << val << x << '\n'; }
  int f1(int);
  void f2();
  C(const char∗ v) { val = v; }
};

using Pmfi = void (C::∗)(int); // pointer to member function of C taking an int
using Pm = const char∗ C::∗; // pointer to char* data member of C

void f(C& z1, C& z2)
{
  C∗ p = &z2;
  Pmfi pf = &C::print;
  Pm pm = &C::val;
  z1.print(1);
  (z1.∗pf)(2);
  z1.∗pm = "nv1 ";
  p−>∗pm = "nv2 ";
  z2.print(3);
  (p−>∗pf)(4);
  pf = &C::f1; // error : retur n type mismatch
  pf = &C::f2; // error : argument type mismatch
  pm = &C::i; // error : type mismatch
  pm = pf; // error : type mismatch
}
```

### Base and Derived Members
We can assign a pointer to a member of a base class, but not the other way around
```c++
class Text : public Std_interface {
public:
  void start();
  void suspend();
  // ...
  virtual void print();
private:
  vector s;
};

void (Std_interface::∗ pmi)() = &Text::print; // error
void (Text::∗pmt)() = &Std_interface::start; // OK
```
