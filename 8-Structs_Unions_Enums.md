# Structures, Unions, Enumerations
_(The C++ Programming Language Chapter 8, ed. 4, by Bjarne Stroustrup)_

## Structures
simple `struct`s are simply a collection of different elements
```c++
struct Address {
  const char∗ name; // "Jim Dandy"
  int number; // 61
  const char∗ street; // "South St"
  const char∗ town; // "New Providence"
  char state[2]; // 'N' 'J'
  const char∗ zip; // "07974"
};

void f()
{
  Address jd = {
    "Jim Dandy",
    61, "South St",
    "New Providence",
    {'N','J'}, "07974"
  };

  jd.name = "Jim Dandy";
  jd.number = 61;
}
```

### struct Layout
members of `struct` are laid out in memory in the order declared

if optimization is necessary, order by largest to smallest in terms of byte usage

### struct Names
`struct`'s name type are available immediately after encountered
```c++
//Cool, works
struct Link {
  Link* previous;
  Link* successor;
};

//Invalid, recursive def
struct No_good {
  No_good member;
};
```

For having multiple dependent `struct`s
```c++
struct List;

struct Link {
  Link* pre;
  Link* suc;
  List* member_of;
  int data;
};

struct List {
  Link* head;
};
```

we can also declare `struct` types with only the name declared, but cannot use the `struct` until it is defined

a `struct` and another type can share a name, don't ever do this
```c++
struct stat {...};
int stat(char* name); //Please hand in your programming license, on your way to the disintegrator
```

### Structures and Classes
a `struct` is just a `class` where the members are `public` by default, so it is valid for `struct`s to have member functions, constructors, etc.

`array` from STL can be used to create arrays of `struct`s
```c++
struct Point {
  int x,y
};

using Array = array<Point,3>; // array of 3 Points

Array points {{1,2},{3,4},{5,6}};
int x2 = points[2].x;
int y2 = points[2].y;
```

### Type equivalence
`struct`s with the same members are different types, and a single member `struct` is not equal to its member type

### Plain Old Data
POD means that an object can be treated as a contiguous sequence of bytes
```c++
struct S0 { }; // a POD
struct S1 { int a; }; // a POD
struct S2 { int a; S2(int aa) : a(aa) { } }; // not a POD (no default constructor)
struct S3 { int a; S3(int aa) : a(aa) { } S3() {} }; // a POD (user-defined default constructor)
struct S4 { int a; S4(int aa) : a(aa) { } S4() = default; }; // a POD
struct S5 { virtual void f(); /\* ... \*/ }; // not a POD (has a virtual function)
struct S6 : S1 { }; // a POD
struct S7 : S0 { int b; }; // a POD
struct S8 : S1 { int b; }; // not a POD (data in both S1 and S8)
struct S9 : S0, S1 {}; // a POD
```

to check if a type is a POD we can use the `is_pod<T>` property predicate defined in `<type_traits>`

### Fields
fields (also called _bit-fields_) are `struct` members that take up a specified number of bits
```c++
struct PPN { // R6000 Physical Page Number
  unsigned int PFN : 22; // Page Frame Number
  int : 3; // unused
  unsigned int CCA : 3; // Cache Coherency Algorithm
  bool nonreachable : 1;
  bool dirty : 1;
  bool valid : 1;
  bool global : 1;
};
```

while this can save memory space, it can increase code size, and it is typically faster to access a `char` or `int`

## Unions
a `union` is a struct in which all the members are allocated at the same address, so only one can be used at once

`union`s can provide performance boosts when used properly, but should be avoided in almost all cases

### Unions and Classes
`union`s are kinds of `struct`s which are kinds of `class`es, but `union`s are restricted from using many of the advanced features available to `class`es like virtual functions, base classes, etc.

### Anonymous unions
```c++
class Entry2 { // two alter native representations represented as a union
private:
  enum class Tag { number, text };
  Tag type; // discriminant

  union { // representation
    int i;
    string s; // string has default constructor, copy operations, and destructor
  };
public:
  struct Bad_entry { }; // used for exceptions

  string name;

  ˜Entry2();
  Entry2& operator=(const Entry2&); // necessary because of the string variant
  Entry2(const Entr y2&);
  // ...

  int number() const;
  string text() const;

  void set_number(int n);
  void set_text(const string&);
  // ...
};

//Example read function
int Entry2::number() const
{
  if (type!=Tag::number) throw Bad_entry{};
  return i;
};

string Entry2::text() const
{
  if (type!=Tag::text) throw Bad_entry{};
  return s;
};

//Example write
void Entry2::set_number(int n)
{
  if (type==Tag::text) {
    s.˜string(); // explicitly destroy str ing (§11.2.4)
    type = Tag::number;
  }
  i = n;
}

void Entry2::set_text(const string& ss)
{
  if (type==Tag::text)
    s = ss;
  else {
    new(&s) string{ss}; // placement new: explicitly construct string (§11.2.4)
    type = Tag::text;
  }
}

//would need additional code for handling destructors and stuff, this is definitely the best thing we could come up with for code efficiency, etc.
```

## Enumerations
Two types of enum
1. `enum class`, the names are local to the `enum` and their values can't be implicitly converted to `int`
2. "Plain" `enum`s, names are in the same scope as `enum` and implicitly convert to `int`

Should almost always use 1.

### enum classes
```c++
enum class Traffic_light { red, yellow, green };
enum class Warning { green, yellow, orange, red };// fire alert levels

Warning a1 = 7; // error : no int->Warning conversion
int a2 = green; // error : green not in scope
int a3 = Warning::green; // error : no Warning->int conversion
Warning a4 = Warning::green; // OK

void f(Traffic_light x)
{
  if (x == 9) { /\* ... \*/ } // error : 9 is not a Traffic_light
  if (x == red) { /\* ... \*/ } // error : no red in scope
  if (x == Warning::red) { /\* ... \*/ } // error : x is not a Warning
  if (x == Traffic_light::red) { /\* ... \*/ } // OK
}
```
by default the _underlying type_ of `enum`s is `int` we can manually define this
```c++
enum class Warning : int { green, yellow, orange, red }; // sizeof(Warning)==sizeof(int)
enum class Warning : char { green, yellow, orange, red }; // sizeof(Warning)==1
```

we can initialize `enum` values with constant expressions of integral type
```c++
enum class Printer_flags {
  acknowledge=1,
  paper_empty=2,
  busy=4,
  out_of_black=8,
  out_of_color=16,

  constexpr Printer_flags operator|(Printer_flags a, Printer_flags b)
  {
    return static_cast<Printer_flags>(static_cast<int>(a))|static_cast<int>(b));
  }
  constexpr Printer_flags operator&(Printer_flags a, Printer_flags b)
  {
    return static_cast<Printer_flags>(static_cast<int>(a))&static_cast<int>(b));
  }
};
```

with the operators we must use explicit conversions because the `enum` is a `enum class`

we use `constexpr` so that we can use the operators with constant expressions, like `switch` statement cases

### Plain enums
don't use them, although you might find in older code
```c++
enum Traffic_light { red, yellow, green };
enum Warning { green, yellow, orang e, red }; // fire alert levels

// error : two definitions of yellow (to the same value)
// error : two definitions of red (to different values)

Warning a1 = 7; // error : no int->Warning conversion
int a2 = green; // OK: green is in scope and converts to int
int a3 = Warning::green; // OK: Warning->int conversion
Warning a4 = Warning::green; // OK

void f(Traffic_light x)
{
  if (x == 9) { / ... / } // OK (but Traffic_light doesn’t have a 9)
  if (x == red) { / ... / } // error : two reds in scope
  if (x == Warning::red) { / ... / } // OK (Ouch!)
  if (x == Traffic_light::red) { / ... / } // OK
}

void q(Traffic_light x)
{
  if (x == red) { / ... / } // OK (ouch!)
  if (x == Warning::red) { / ... / } // OK (ouch!)
  if (x == Traffic_light::red) { / ... / } // error : red is not a Traffic_light value
}
```

### Unnamed enums
Only plain `enum`s can be unnamed
```c++
enum {arrow_up=1, arrow_down, arrow_sideways};
```

can be used to setup integer constants
