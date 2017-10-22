_(The C++ Programming Language Chapter 2, ed. 4, by Bjarne Stroustrup)_

sizeof(type) - returns number of bytes for type size

---

Assignment `int i1 = 7.2;` (cast) `int i2 {7.2}` (error)

_can use `=` and `{}` together, but it is redundant to do so_

---

`auto` infers type, use with `=`
```c++
auto b = true; // a bool
auto ch = 'x'; // a char
auto i = 123; // an int
auto d = 1.2; // a double
auto z = sqrt(y); // z has the type of whatever sqr t(y) returns
```

---

`const` value will not be changed, compiler enforced

`constexpr` evaluated at compile time, will not change
- _functions can only be used with this if they are also declared with `constexpr` and consist of a simple return statement_

---

`<<` output operator, puts right side into left

`>>` input operator, puts left side into right
```c++
std::cout << "Hello";

char reply = 0;
std::cin >> reply;
```

-

```c++
switch (answer) {
  case 'y':
    return true;
  case 'n':
    return false;
  default:
    cout << "Sorry, I don't understand that.\n";
    ++tries; // increment
}
```

---

`*` = "contents of"

`&` = "address of"

```c++
char v[6]; //Array of six chars

char* p = &v[3]; //p points to v's fourth elem
char x = *p; //*p is the object the p points to
```

---

```c++
//Ranged for-loops
int v[] = {0,1,2,3,4,5,6,7,8,9};

for (auto x : v) // for each x in v
  cout << x << '\n';
for (auto x : {10,21,32,43,54,65})
  cout << x << '\n';

for (auto& x : v)
  ++x;
```

in a declaration unary `&` means reference to, don't need `*` prefix to access contents

```c++
T a[n]; // T[n]: array of n Ts (§7.3)
T∗ p; // T*: pointer to T (§7.2)
T& r; // T&: reference to T (§7.7)
T f(A); // T(A): function taking an argument of type A returning a result of type T (§2.2.1)
```

---

```c++
nullptr // Represents nullptr

if(p == nullptr) return 0; //Simple nullptr check
```

### Pointers vs. References

1. A pointer can be re-assigned any number of times while a reference cannot be re-seated after binding.
1. Pointers can point nowhere (NULL), whereas references always refer to an object.
1. You can't take the address of a reference like you can with pointers.
1. There's no "reference arithmetics" (but you can take the address of an object pointed by a reference and do pointer arithmetics on it as in `&obj + 5`).


To clarify a misconception:


The C++ standard is very careful to avoid dictating how a
compiler must implement references,
but every C++ compiler implements references as pointers. That is, a declaration such as:
```c++
 int &ri = i;
```
if  not optimized away entirely, allocates the same amount of storage as a pointer, and places the address of i into that storage.


__*So, a pointer and a reference both occupy the same amount of memory.*__

As a general rule,

- Use references in function parameters and return types to define useful and self-documenting interfaces.
- Use pointers to implement algorithms and data structures.

-

`new` allocates memory from _free store_ (aka _dynamic memory_ or _heap_)

## User Defined Types, Basics

`struct`

```c++
struct Vector {
  int sz; // number of elements
  double∗ elem; // pointer to elements
};

void vector_init(Vector& v, int s)
{
  v.elem = new double[s]; // allocate an array of s doubles
  v.sz = s;
}

void f(Vector v, Vector& rv, Vector∗ pv)
{
  int i1 = v.sz; // access through name
  int i2 = rv.sz; // access through reference
  int i4 = pv−>sz; // access through pointer
}
```

`class` + `operator`

```c++
class Vector {
public:
  Vector(int s) :elem{new double[s]}, sz{s} { } // construct a Vector
  double& operator[](int i) { return elem[i]; } // element access: subscripting
  int size() { return sz; }
private:
  double∗ elem; // pointer to the elements
  int sz;       // the number of elements
}
```

`enum`

```c++
enum class Color { red, blue , green };
enum class Traffic_light { green, yellow, red };

Color x = red; // error : which red?
Color y = Traffic_light::red; // error : that red is not a Color
Color z = Color::red; // OK
```

> If you don’t want to explicitly qualify enumerator names and want enumerator values to be ints (without the need for an explicit conversion), you can remove the `class` from `enum class` to get a ‘‘plain `enum`’’


Example of `enum class` operator overloading
```c++
Traffic_light& operator++(Traffic_light& t)
// prefix increment: ++
{
  switch (t) {
    case Traffic_light::green: return t=Traffic_light::yellow;
    case Traffic_light::yellow: return t=Traffic_light::red;
    case Traffic_light::red: return t=Traffic_light::green;
  }
}
Traffic_light next = ++light; // next becomes Traffic_light::green
```


## Basic Templates

```c++
// Vector.h:
class Vector {
public:
  Vector(int s);
  double& operator[](int i);
  int size();
private:
  double∗ elem; // elem points to an array of sz doubles
  int sz;
};
```

```c++
// Vector.cpp:
#include "Vector.h" // get the interface
Vector::Vector(int s)
  :elem{new double[s]}, sz{s}
{
}
double& Vector::operator[](int i)
{
  return elem[i];
}
int Vector::siz e()
{
  return sz;
}
```

---

`namespaces`

```c++
namespace My_code {
  class complex {
    // ...
  };
  complex sqr t(complex);
  // ...
  int main();
}

int My_code::main()
{
  complex z {1,2};
  auto z2 = sqrt(z);
  std::cout << '{' << z2.real() << ',' << z2.imag() << "}\n";
  // ...
};

int main()
{
  return My_code::main();
}
```

---

### Exceptions
```c++
throw out_of_range{"Vector::operator[]"};
```
note that out_of_range is specified in the standard library

---

`static_assert` throws an error at compile time if condition not met
```c++
static_assert(4<=sizeof(int), "integers are too small"); // check integer size
```
