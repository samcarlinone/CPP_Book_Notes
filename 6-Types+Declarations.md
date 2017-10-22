# Types and Declarations
_(The C++ Programming Language Chapter 6, ed. 4, by Bjarne Stroustrup)_

Some things like number of bits used to store a `char` may vary by implementation, leading to undefined behavior in some cases

undefined behavior is when individual implementations can produce different outcomes

### Implementations
_hosted_ implementations provide all standard library facilities, while _freestanding_ implementations provide a smaller subset

### bool(s)
true converts to the number 1, false converts to the number 0

when going from number to `bool`, any non-zero number is true

this behavior can be changed using brace-initializers `bool b {i>0};`

when used with pointers `nullptr` is false, anything else is true

### char (and family)
- `char`: The default character type, used for program text. A `char` is used for the implementation’s character set and is usually 8 bits.
- `signed char`: Like `char`, but guaranteed to be signed, that is, capable of holding both positive and negative values.
- `unsigned char`: Like `char`, but guaranteed to be unsigned.
- `wchar_t`: Provided to hold characters of a larger character set such as Unicode (see §7.3.2.2). The size of wchar_t is implementation-defined and large enough to hold the largest character set supported by the implementation’s locale (Chapter 39).
- `char16_t`: A type for holding 16-bit character sets, such as UTF-16.
- `char32_t`: A type for holding 32-bit character sets, such as UTF-32

`signed char`, `unsigned char`, and `char` are all different types, but the implementation will have plain `char` be signed or unsigned

This can lead to undefined behavior

### character literals
denoted by `''`, has type `char`

multi-character literals should be avoided, `'foo'` __BAD__

### Declaring Multiple Names
```c++
int* p, q;
//is the same as
int* p; int q; // NOT int* p; int* q;
```

### Scope
`{}` defines a local scope

```c++
int x; // global x
void f()
{
  int x; // local x hides global x
  x = 1; // assign to local x
  {
    int x; // hides first local x
    x = 2; // assign to second local x
  }
  x = 3; // assign to first local x
}
int∗ p = &x; // take address of global x
```

### Initialization
```c++
X a1 {v};  //Preferred
X a2 = {v};
X a3 = v;
X a4(v);
```

one pitfall, when using `auto` with `{}` type can be `initializer list`, when used with multiple values

## Objects and Values
### Lvalues and Rvalues

"__m__" is movable, and "__i__" is has identity

lvalue {__i__,!__m__} <- glvalue {__i__} -> xvalue {__i__,__m__} <- rvalue {__m__} -> prvalue {!__i__,__m__}

## Type Aliases
```c++
using Pchar = char*; //*
using PF = int(*)(double); //*

Pchar p1 = nullptr;
char* p3 = p1; //fine

typedef int int32_t; // equivalent to "using int32_t = int;"

using Char = char;
using Uchar = unsigned Char; // error
using Uchar = unsigned char; // OK
```
