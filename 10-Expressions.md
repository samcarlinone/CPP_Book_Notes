# Expressions
_(The C++ Programming Language Chapter 10, ed. 4, by Bjarne Stroustrup)_

Most of the chapter is taken up with an example program I will not reproduce here

### Operator Precedence
Use parens to ensure proper order of operations

### Temporary Objects
Temporary objects may be created, be aware of objects deleting their resources
```c++
void f(string& s1, string& s2, string& s3)
{
  const char∗ cs = (s1+s2).c_str();
  cout << cs;
  if (strlen(cs=(s2+s3).c_str())<8 && cs[0]=='a') {
    // cs used here
  }
}
```
In this case a temporary `string` object is created to hold s1+s2, then a pointer is retrieved by using `c_str()`.

However the object produced s1+s2 is destroyed after the expression is completed(by the next line), and so will automatically deallocate the array produced by `c_str()`. The attempt to print `cs` is likely to result in undefined behaviors and strange bugs.

## Constant Expressions
`constexpr` is a way of designating expressions that should be evaluated at compile time

within these expressions we can use any operator that doesn't modify state

like `const` these expressions help avoid magic numbers and repetitive declarations

## Implicit Type Conversion
### Promotions
promotions are when a type is converted into a "larger" type, these conversions lose no data

### Conversions
You can also perform narrowing conversions that lose data
```c++
void f(double d) {
  char c = d; //Allowed, but, well, why?
}
```

Many other conversions can occur, google it for your specific case
