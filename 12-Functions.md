# Functions
_(The C++ Programming Language Chapter 12, ed. 4, by Bjarne Stroustrup)_

### Parts of a Function Declaration
- the name of the function
- the argument list, may be empty `()`
- the return type which can be `void` or be prefixed or suffixed
- `inline` indicating a desire to inline calls to the function
- `constexpr` indicating the function should be evaluatable at compile time
- `noexcept` indicating the function may not throw an exception
- a linkage specification such as `static`
- `[[noreturn]]` indicating the function willl not return using the normal call / return mechanism

In addition a function may be specified as
- `virtual`, indicating it can be overridden in derived classes
- `override`, indicating it is overriding a `virtual` function
- `final`, indicating it cannot be overridden
- `static`, indicating not associated with a particular object
- `const`, indicating it may not modify it's object

### Returning Values
```c++
string to_string(int a); // prefix return type
auto to_string(int a) −> string; // suffix return type
```
suffix return type used with templates
```c++
template<class T, class U>
auto product(const vector<T>& x, const vector<U>& y) −> decltype(x∗y);
```

### inline Functions
code is copied to calling location, rather then generated once and used elsewhere

this can increase performance, but to the detriment of compiled size

### constexpr Functions
by designating a function `constexpr` we can call it from other `constexpr`

### [[noreturn]] Functions
`[[...]]` called an attribute and used to specify implementation dependent properties

useful for code that will not return
```c++
[[noreturn]] void exit(int);
```

### Local Variables
local variables may be declared `static` to become available to every call of the function, without introducing global variables

## Argument Passing
Arguments can be passed by reference, using `&`

This should often by used with `const` to show that the function will not modify the passed argument, and is simply avoiding an expensive copy

### Array Arguments
when arrays are passed, they are passed as a pointer to the initial element
```c++
void odd(int∗ p);
void odd(int a[]); // the same
void odd(int buf[1020]); // the same
```

C-style strings are 0 terminated, so `strlen()` can find their length, in general however a second size argument is passed
```c++
void compute1(int∗ vec_ptr, int vec_size); // one way
```

it is also possible to pass a reference to an array
```c++
void f(int(&r)[4]);

void g()
{
  int a1[] = {1,2,3,4};
  int a2[] = {1,2};
  f(a1); // OK
  f(a2); // error : wrong number of elements
}
```

while sometimes necessary for template programming, it is best avoided in favor of `vector` and the like

### List Arguments
a `{}` initializer list can be passed as a parameter
```c++
template<class T>
void f1(initializer_list<T>);

struct S {
  int a;
  string s;
};

void f2(S);

template<class T, int N>
void f3(T (&r)[N]);

void f4(int);

void g()
{
  f1({1,2,3,4}); // T is int and the initializer_list has size() 4
  f2({1,"MKS"}); // f2(S{1,"MKS"})
  f3({1,2,3,4}); // T is int and N is 4
  f4({1}); // f4(int{1});
}
```

### Unspecified Number of Arguments
when we don't know the number of arguments we have several options
- variadic templates
- `initializer_list` type arguments
- end the argument list with `...`, not type safe, but has been around a while

```c++
int printf(const char∗ ...);


//The following will break during run time
#include <cstdio>
int main()
{
  std::printf("My name is %s %s\n",2);
}
```

### Default Arguments
```c++
int f(int, int =0, char∗ =nullptr); // OK
int g(int =0, int =0, char∗); // error
int h(int =0, int, char∗ =nullptr); // error

int nasty(char*=nullptr) // error
```

## Overloading Functions
Very useful in some cases

### Automatic Overload Resolution
```c++
void print(double);
void print(long);

void f()
{
  print(1L); // print(long)
  print(1.0); // print(double)
  print(1); // error, ambiguous: print(long(1)) or print(double(1))?
}
```

note that if an exact match is not found, various conversions and casts may be used

1. Exact Match
2. Match using promotions; (`bool` to `int`, etc.)
3. Match using standard conversions (`int` to `double`, etc.)
4. Match using user-defined conversions (`double` to `complex<double>`)
5. Math using the `...` in a function declaration

### Overloading and Return Type
Return types are not considered in overload resolution

Probably not a good idea to return different types, use different function names in that case

### Overloading and Scope
By default the functions will only overload function in the same namespace scope

### Resolution for Multiple Arguments
tries to find one that is the best fit
```c++
int pow(int, int);
double pow(double , double);
complex pow(double , complex);
complex pow(complex, int);
complex pow(complex, complex);

void k(complex z)
{
  int i = pow(2,2); // invoke pow(int,int)
  double d = pow(2.0,2.0); // invoke pow(double,double)
  complex z2 = pow(2,z); // invoke pow(double,complex)
  complex z3 = pow(z,2); // invoke pow(complex,int)
  complex z4 = pow(z,z); // invoke pow(complex,complex)
}

void g()
{
  double d = pow(2.0,2); // error : pow(int(2.0),2) or pow(2.0,double(2))?
}
```

### Manual Overload Resolution
```c++
void f1(char);
void f1(long);
void f2(char∗);
void f2(int∗);

void k(int i)
{
  f1(i); // ambiguous: f1(char) or f1(long)?
  f2(0); // ambiguous: f2(char*) or f2(int*)?
}
```

we could manually cast, but it would probably be better to evaluate whether our functions could be redesigned

## Pre- and Postconditions
```c++
int area(int len, int wid)
/*
calculate the area of a rectangle
precondition: len and wid are positive
postcondition: the return value is positive
postcondition: the return value is the area of a rectange with sides len and wid
*/
{
return len∗wid;
}
```

who is in charge of ensuring these conditions are met, caller or implementer?

not a simple question, more discussion later

## Pointer to Function
```c++
void error(string s) { ... }
void (∗efct)(string); // pointer to function taking a string argument and returning nothing

void f()
{
  efct = &error; // efct points to error
  efct("error"); // call error through efct
}
```

dereferencing and referencing is optional
```c++
void (∗f1)(string) = &error; // OK: same as = error
void (∗f2)(string) = error; // OK: same as = &error

void g()
{
  f1("Vasa"); // OK: same as (\*f1)("Vasa")
  (∗f1)("Mary Rose"); // OK: as f1("Mary Rose")
}
```

we can also cast pointers
```c++
using P1 = int(∗)(int∗);
using P2 = void(∗)(void);

void f(P1 pf)
{
  P2 pf2 = reinterpret_cast<P2>(pf)
  pf2(); // likely serious problem
  P1 pf1 = reinterpret_cast<P1>(pf2); // convert pf2 "back again"
  int x = 7;
  int y = pf1(&x); // OK
  // ...
}
```

in many cases lambdas can provide a cleaner alternative to function pointers

A pointer to a function must have the same linkage specs as the original, thus neither linkage specs nor `noexcept` can appear in type aliases and should be avoided in code
```c++
void f(int) noexcept;
void g(int);

void (∗p1)(int) = f; // OK: but we throw away useful information
void (∗p2)(int) noexcept = f; // OK: we preserve the noexcept information
void (∗p3)(int) noexcept = g; // error : we don’t know that g doesn’t throw

//using Pc = extern "C" void(int); // error : linkage specification in alias
using Pn = void(int) noexcept; // error : noexcept in alias
```

## Macros
used a lot in C, should be avoided when possible in C++
