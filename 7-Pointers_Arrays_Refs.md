# Pointers, Arrays, and References
_(The C++ Programming Language Chapter 7, ed. 4, by Bjarne Stroustrup)_

## Pointers

For a type `T`, `T*` is the type "pointer to `T`"
```c++
char c = 'a';
char* p = &c;

//* is the dereference operator

char c2 = *p;
```

some pointers are more complex
```c++
int∗ pi; // pointer to int
char∗∗ ppc; // pointer to pointer to char
int∗ ap[15]; // array of 15 pointers to ints
int (∗fp)(char∗); // pointer to function taking a char* argument; returns an int
int∗ f(char∗); // function taking a char* argument; returns a pointer to int
```
### void*
`void*` = "pointer to an object of unknown type"

```c++
void f(int∗ pi)
{
  void∗ pv = pi; // ok: implicit conversion of int* to void*
  ∗pv; // error : can’t dereference void*
  ++pv; // error : can’t increment void* (the size of the object pointed to is unknown)

  int∗ pi2 = static_cast<int∗>(pv); // explicit conversion back to int*

  double∗ pd1 = pv; // error
  double∗ pd2 = pi; // error
  double∗ pd3 = static_cast<double∗>(pv); // unsafe (§11.5.2)
}
```

usage of `void*` should be avoided, except for low level system programming

### nullptr
`nullptr` represents a pointer that isn't pointing to an object, `nullptr` cannot be assigned to a non-pointer variable

## Arrays
For a type `T`, `T[size]` is the type "array of `size` elements of type `T`", with indexes from [0, `size`-1]

Attempting to access out of bounds array elements is undefined behavior and potentially disastrous

### Array Initializers
```c++
char v2[] = {'a', 'b', 'c', 0}; //Ok
char v3[2] = {'a', 'b', 'c'}; //error: too many initializers

//empty elements assigned 0
int v5[8] = {1, 2, 3, 4};
//is the same as
int v5[8] = {1, 2, 3, 4, 0, 0, 0, 0};
```

arrays do not have built in copy constructors, and cannot be passed by value

### String Literals
A _string literal_ is a character sequence enclosed in double quotes: `"this is a string"`

A string literal has one char at the end `\0` to signify the end of the string

```c++
const char∗ p = "Heraclitus";
const char∗ q = "Heraclitus";

void g()
{
  if (p == q) cout << "one!\n"; // the result is implementation-defined
  // ...
}
```

Long strings can be broken across lines
```c++
char alpha[] = "Hello "
               "me llamo"
               "Bob";
```

#### Raw Character Strings
`\` is used in string literals to represent various chars, however for things like regexes it can become awkward to work with

```c++
string s = "\\w\\\\w"; //Annoying
string s = R"(\w\\w)"; //Noice

//but what if we wanted to use )" in the string?

R"***("quoted string containing the usual terminator ("))")***"
//"quoted string containing the usual terminator ("))"

// the character patter before the ( must match the one after )
```

#### Larger Character Sets
A string with the prefix `L` e.g. `L"magic"` us a wide char string `const wchar_t[]`, can be combined with `R` for large raw string

```c++
"folder\\file" // implementation character set string
R"(folder\file)" // implementation character raw set string
u8"folder\\file" // UTF-8 string
u8R"(folder\file)" // UTF-8 raw string
u"folder\\file" // UTF-16 string
uR"(folder\file)" // UTF-16 raw string
U"folder\\file" // UTF-32 string
UR"(folder\file)" // UTF-32 raw string

//use \u for unicode codepoint
u8"Codepoint: \u00E6"
```

`u` must always precede `R`

## Pointers into arrays
Arrays are easily converted into pointers
```c++
int v[] = {1,2,3,4};
int* p1 = v; //Pointer to initial element
int* p2 = &v[0]; //Also pointer to initial element
int* p3 = v+4; //pointer to one-beyond-last element
```

when arrays are passed to functions as pointers they lose their size. `strlen(p)` counts to the terminating `0`

### Navigating Arrays
```c++
void fi(char v[])
{
  for (int i = 0; v[i]!=0; ++i)
    use(v[i]);
}

void fp(char v[])
{
  for (char∗ p = v; ∗p!=0; ++p)
    use(∗p);
}
```

pointer subtraction is only allowed when both pointers are items in an array (although c++ has no way to stop you if they aren't), and addition of pointers is disallowed completely

### Multidimensional Arrays
Internally converted to flat arrays, comma notation not allowed

### Passing Arrays
```c++
void comp(double arg[10]) // arg is a double*
{
  for (int i=0; i!=10; ++i)
    arg[i]+=99;
}

void f()
{
  double a1[10];
  double a2[5];
  double a3[100];
  comp(a1);
  comp(a2); // disaster!
  comp(a3); // uses only the first 10 elements
};
```

not copied, works on existing arrays, array size is not verified in any way

## Pointers and const
```c++
char ∗const cp; // const pointer to char
char const∗ pc; // pointer to const char
const char∗ pc2; // pointer to const char
```

using explicit conversions, `const`-ness can be bypassed, if you want to watch the world burn

## Pointers and Ownership
Any code can delete pointers, but if other pointers still reference object undefined behavior occurs

Use resource handle classes like `unique_ptr`

## References
Essentially aliases for objects, cannot be changed to reference a different object, never null, no need to use dereference operator

3 types of reference
- lvalue reference: a reference to an object we can change
- `const` reference: a reference to an object we cannot change
- rvalue reference: refers to an object we do not need to preserve (temporary)

### Lvalue References
`X&` means "reference to `X`"

references can be thought of like const pointers, but it is important to remember they are not their own objects like a pointer

### Rvalue References
`T&&` means "reference to temporary object `T`"

```c++
template<class T>
void swap(T& a, T& b) // "perfect swap" (almost)
{
  T tmp {static_cast<T&&>(a)}; // the initialization may write to a
  a = static_cast<T&&>(b); // the assignment may write to b
  b = static_cast<T&&>(tmp); // the assignment may write to tmp
}
```

in this example the temporary references allow the compiler to use move rather than copy operations

### References to References
this can only happen as a result of an alias or template type argument
```c++
using rr_i = int&&;
using lr_i = int&;
using rr_rr_i = rr_i&&; // ‘‘int && &&’’ is an int&&
using lr_rr_i = rr_i&; // ‘‘int && &’’ is an int&
using rr_lr_i = lr_i&&; // ‘‘int & &&’’ is an int&
using lr_lr_i = lr_i&; // ‘‘int & &’’ is an int&
```

lvalue is almost always the result

### Pointers and References
Pointers can be used for arrays of objects, and changing referenced object, can also be null

References are always safe to use, can specify usage
