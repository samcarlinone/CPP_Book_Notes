# Namespaces
_(The C++ Programming Language Chapter 14, ed. 4, by Bjarne Stroustrup)_

```c++
namespace Graph_Lib {
  class Shape { ... };
  class Line : public Shape { ... };

  Shape operator+(const Shape&, const Shape&); //compose
}
```
