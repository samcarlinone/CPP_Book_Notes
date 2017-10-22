# Select Operations
_(The C++ Programming Language Chapter 11, ed. 4, by Bjarne Stroustrup)_

### Logical Operators
like in other languages `&&` and `||` only evaluate the second statement if necessary

### Bitwise Logical Operators

Operator | Type
--- | ---
`&` | And
&#124; | Or
`^` | Xor
`~` | Complement
`>>` | Right Shift
`<<` | Left Shift

## Free Store
`new` and `delete` allow the programmer to allocate memory from the free store (or heap, or dynamic memory)

C++ __does not guarantee garbage collection__

### Memory Management
3 main problems when using free store
- Leaked objects: People use `new` and forget to call `delete`
- Premature deletion: People use `delete`, then later try to use a pointer to the delete object
- Double deletion: An object is deleted twice

2 main techniques for avoiding these pitfalls
- Prefer scoped variables to free store
- Put pointers into manager objects like `shared_ptr`, `unique_ptr`, etc.

### Arrays
The `new` operator is used to create arrays and `delete[]` is used to delete arrays

__Remember: things like errors or returns can cause the `delete` operator to be skipped, even if it is present__

### Getting Memory Space
in general c++ may not error, even when physical memory is exhausted leading to high virtual memory usage and potentially severe issues, in some cases the `bad_alloc` error may be thrown

### Overloading new
```c++
void∗ operator new (size_t sz, void∗ p) noexcept; // place object of size sz at p
void∗ operator new[](size_t sz, void∗ p) noexcept; // place object of size sz at p

void operator delete (void∗ p, void∗) noexcept; // if (p) make *p invalid
void operator delete[](void∗ p, void∗) noexcept; // if (p) make *p invalid
```

probably best left alone for the most part

### nothrow new
```c++
void f(int n)
{
  int∗ p = new(nothrow) int[n]; // allocate n ints on the free store
  if (p==nullptr) {// no memory available
    // ... handle allocation error ...
  }
  // ...
  operator delete(nothrow,p); // deallocate \*p
}
```

functions of form
```c++
void∗ operator new(size_t sz, const nothrow_t&) noexcept; // allocate sz bytes;
// return nullptr if allocation failed
void operator delete(void∗ p, const nothrow_t&) noexcept; // deallocate space allocated by new
void∗ operator new[](size_t sz, const nothrow_t&) noexcept; // allocate sz bytes;
// return nullptr if allocation failed
void operator delete[](void∗ p, const nothrow_t&) noexcept; // deallocate space allocated by new
```

returns `nullptr` rather than throwing `bad_alloc`

## Lists
```c++
struct S { int a, b; };
struct SS { double a, b; };

void f(S); // f() takes an S
void g(S);
void g(SS); // g() is overloaded

void h()
{
  f({1,2}); // OK: call f(S{1,2})
  g({1,2}); // error : ambiguous
  g(S{1,2}); // OK: call g(S)
  g(SS{1,2}); // OK: call g(SS)
}
```

initializer lists use pass by value and copy operations with elements, fine for values like `int` but potentially costly with more expensive constructs

## Lambda Expressions
_lambda expressions_ consist of several parts
- possibly empty capture list delimited by `[]`
- optional parameter list delimited by `()`
- an optional `mutable` specifier that indicates the lambda will modify captured variables
- an optional `noexcept` specifier
- an optional return type declaration of the form `->` _type_
- a body delimited by `{}`

### Alternatives to lambdas
```c++
void print_modulo(const vector<int>& v, ostream& os, int m)
// output v[i] to os if v[i]%m==0
{
  class Modulo_print {
    ostream& os; // members to hold the capture list
    int m;
  public:
    Modulo_print (ostream& s, int mm) :os(s), m(mm) {} // capture
    void operator()(int x) const
      { if (x%m==0) os << x << '\n'; }
  };

  for_each(begin(v),end(v),Modulo_print{os,m});
}
```
compared to
```c++
void print_modulo(const vector<int>& v, ostream& os, int m)
// output v[i] to os if v[i]%m==0
{
  auto Modulo_print = [&os,m] (int x) { if (x%m==0) os << x << '\n'; };

  for_each(begin(v),end(v),Modulo_print);
}
```

### Lambda and Lifetime
if a lambda is going to be passed or executed later, be sure to only capture by value to prevent invalid references

### Lambda Type
```c++
auto rev = [&rev](char∗ b, char∗ e)
  { if (1<e−b) { swap(∗b,∗−−e); rev(++b,e); } }; // error, can't use rev, type hasn't been deduced

function<void(char∗ b, char∗ e)> rev =
  [&](char∗ b, char∗ e) { if (1<e−b) { swap(∗b,∗−−e); rev(++b,e); } };
```

## Explicit Type Conversion
- Construction using `{}`
- Specific casts
  - `const_cast`
  - `static_cast`
  - `reinterpret_cast`
  - `dynamic_cast`
- C-style casts
- Functional notation

`{}` will only perform "well-behaved" conversions

even `int` to `double` is disallowed because they might both have the same number of bits, which would result in data loss

### C-style casts
avoid `(T) e`, as it may not be portable or result in data loss
