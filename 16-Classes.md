# Classes
_(The C++ Programming Language Chapter 16, ed. 4, by Bjarne Stroustrup)_

### Class and Struct
struct = public by default
class = private by default

### Mutability
`const` functions do not modify object state

for an internal variable that doesn't affect external state we can use `mutable`

### Self-Reference
```c++
class BankAccount {
  BankAccount& deposit (int ammount) {
    // ...
    return *this; //*
  }
}

BankAccount mine;
mine.deposit(10).deposit(100);
```

### static Members
```c++
class Date {
  int d, m, y;
  static Date default_date;
public:
  Date(int dd =0, int mm =0, int yy =0);
  // ...
  static void set_default(int dd, int mm, int yy); // set default_date to Date(dd,mm,yy)
};
```

default_date shared by all instances
