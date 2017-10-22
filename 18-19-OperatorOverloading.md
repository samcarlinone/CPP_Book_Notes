# Operator Overloading
_(The C++ Programming Language Chapter 18 & 19, ed. 4, by Bjarne Stroustrup)_

We can overload almost every operator

### Unary Operators
```c++
class X {
  public: // members (with implicit this pointer):
  X∗ operator&(); // prefix unary & (address of)
  X operator&(X); // binary & (and)
  X operator++(int); // postfix increment (see §19.2.4)
  X operator&(X,X); // error : ternary
  X operator/(); // error : unary /
};
```

### Predefined meaning
While `++` is equivalent too `a+=1` which is equivalent too `a=a+1` the computer will __not__ generate a definition for `+=` from the definitions of `+` and `=`

### Operators and User-Defined Types
an operator must either be a member or take at least one argument of a user-defined type, thus the language itself cannot be changed

### User Defined Literals
This let's us define user class based literals

```c++
complex operator+(complex a, complex b)
{
  return a += b; // access representation through +=
} //Overrides the + operator globally for use with user created, complex class
```

### Literals
```c++
constexpr complex<double> operator "" i(long double d) // imaginary literal
{
  return {0,d}; // complex is a literal type
}

complex z1 {1.2+12e3i};
complex f(double d)
{
  auto x {2.3i};
  return x+sqrt(d+12e3i)+12e3i;
}
```

# Chapter 19 - Advanced Operators
You can overload new and delete, only do this if you want to cry
