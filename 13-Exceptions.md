# Exception Handling
_(The C++ Programming Language Chapter 13, ed. 4, by Bjarne Stroustrup)_

The RAII technique means that the resource handle object will deallocate the memory properly, even if an exception interrupts normal program flow

### Rethrow
Sometimes an exception handler cannot completely, in this case the exception can be rethrown

```c++
else {
  throw; //Throw without an operand indicates a rethrow
}
```

### Catching Exceptions
```c++
void f()
{
  try {
    throw E{};
  }
  catch(H) {
    //Handle Error
  }
}
```

1. If `H` is the same type as `E`
2. If `H` is an unambiguous public base of `E`
3. If `H` and `E` are pointer types or 1. or 2. holds for the types to which they refer
4. If `H` is a reference and 1. or 2. holds for the type to which `H` refers

### Catch Any Exception
```c++
catch (...) { // handle every exception
  // ... cleanup ...
  throw;
}
```
