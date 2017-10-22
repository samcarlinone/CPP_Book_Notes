# Derived Classes
_(The C++ Programming Language Chapter 22, ed. 4, by Bjarne Stroustrup)_

### dynamic_cast
`dynamic_cast<T*>(p)` will return a pointer of type `T*` or `nullptr` if `p` is not of the correct type

`dynamic_cast` requires a pointer or a reference to a polymorphic type in order to do a down or crosscast

A polymorphic types is a class with at least one virtual function

#### dynamic_cast to Reference
since we cannot have null references, if the operand cannot be converted a `bad_cast` exception will be thrown

### Multiple Inheritance

```c++
class Component
  : public virtual Storable { };
class Receiver
  : public Component { };
class Transmitter
  : public Component { };
class Radio
  : public Receiver, public Transmitter { };

void h1(Radio& r)
{
  Storable∗ ps = &r; // a Radio has a unique Storable
  // ...
  Component∗ pc = dynamic_cast<Component∗>(ps); // pc = 0; a Radio has two Components
  // ...
}

void h2(Storable∗ ps) // ps might or might not point to a Component
{
  if (Component∗ pc = dynamic_cast<Component∗>(ps)) {
    // we have a component!
  }
  else {
    // it wasn’t a Component
  }
}
```

### static_cast and dynamic_cast
```c++
void g(Radio& r)
{
  Receiver∗ prec = &r; // Receiver is an ordinary base of Radio
  Radio∗ pr = static_cast<Radio∗>(prec); // OK, unchecked
  pr = dynamic_cast<Radio∗>(prec); // OK, run-time checked

  Storable∗ ps = &r; // Storable is a virtual base of Radio
  pr = static_cast<Radio∗>(ps); // error : cannot cast from virtual base
  pr = dynamic_cast<Radio∗>(ps); // OK, run-time checked
}
```

## Type Identification
`typeid` returns a `type_info` object containing information about the class

```c++
void f(Non_poly& npr, Poly& pr)
{
  cout << typeid(npr).name() << '\n'; // writes something like "Non_poly"
  cout << typeid(pr).name() << '\n'; // name of Poly or a class derived from Poly
}
```
