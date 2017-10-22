# Source Files and Programs
_(The C++ Programming Language Chapter 15, ed. 4, by Bjarne Stroustrup)_

### File-Local Names

avoid global vars, if necessary either place var in unnamed namespace, or declare `static`

`static` means use internal linkage

### Header Files
`#include` _header files_, use `<` and `>` instead of quotes for standard libraries

Headers should never include

Name | Example
---  | ---
Ordinary function definitions | `char get(char* p){return *p++; }`
Data definitions | `int a;`
Aggregate definitions | `short tbl[] = { 1, 2, 3 };`
Unnamed namespaces | `namespace { /* ... */ }`
`using`-directives | `using namespace Foo;`

Header files typically use the extension `.h`, while source files use `.cpp`

### The One-Definition Rule
A given class, enum, template, etc. must be defined exactly once

Of course it is more complex then that, and so we have _the one-definition rule_ or "the ODR"

It says that two definitions are the same iff
1. they appear in different translation units, and
2. they are token-for-token identical, and
3. the meanings of those tokens are the same in both translation units

```c++
// s.h:
struct S { int a; char b; };
void f(S∗); //*
// file1.cpp:
#include "s.h"
// use f() here
// file2.cpp:
#include "s.h"
//void f(S∗ p) { /* ... */ } //*
```

### Linkage to Non-C++ Code
```c++

extern "C" stuff(); //"
extern "C" { //"
  stuff();

  //Or even a full header
  #include <cOnly.h>
}
```
