# Standard Library Facilities
_(The C++ Programming Language Chapter 4, ed. 4, by Bjarne Stroustrup)_

### Libraries
The STL is similar to the C Standard Library, with a few changes for type safety

Can `#include <*>` specific facilities e.g. `#include <list>`

can also use `using namespace std;` however this is frowned upon as it includes many, likely unneeded, modules.

The prefix `std::` or `#include <*>` are recommended

Selected Standard Library Headers | Notable Inclusions
--- | ---
`<algorithm>`      | copy(), find(), sort()
`<array>`          | array
`<chrono>`         | duration, time_point
`<cmath>`          | sqrt(), pow()
`<complex>`        | complex, sqrt(), pow()
`<fstream>`        | fstream, ifstream, ostream
`<future>`         | future, promise
`<iostream>`       | istream, ostream, cin, cout
`<map>`            | map, multimap
`<memory>`         | unique_ptr, shared_ptr, allocator
`<random>`         | default_random_engine, normal_distribution
`<regex>`          | regex, smatch
`<string>`         | string, basic_string
`<set>`            | set, multiset
`<sstream>`        | istream, ostream
`<thread>`         | thread
`<unordered_map>`  | unordered_map, unordered_multimap
`<utility>`        | move(), swap(), pair
`<vector>`         | vector

### Strings

can be concatenated like normal,


### IO
`cout` and `cin` with `<<` and `>>` (these are operators that can be implemented on one's own classes)

```c++
string str;
getline(cin, str);

//Reads a whole line, \n and all
```

Overriding `<<` and `>>`
```c++
ostream& operator<<(ostream& os, const Entry& e)
{
  return os << "{\"" << e.name << "\", " << e.number << "}";
}

istream& operator>>(istream& is, Entry& e)
{
  char c;
  is >> c;

  string name; // the default value of a string is the empty string: ""
  while (is.get(c) && c!='"')
    name+=c;

  if(fail)
    is.setf(ios_base::failbit); // register the failure in the stream

  return is;
}
```

## Containers

### vector
can be initialized with a set of values (`{1,2,3}`), and accessible through subscripting (`[]`)

```c++
vector<int> v1 = {1, 2, 3, 4}; // size is 4
vector<string> v2; // size is 0
vector<Shape∗> v3(23); // size is 23; all slots filled with nullptr
vector<double> v4(32,9.9); // size is 32; all slots filled with 9.9
```

supports the `for(auto x: list)` syntax

`push_back` allows easy appending to `vector`

`at()` implements range checking, or use `[]` and catch `out_of_range` exception

### list
Doubly-linked list, allows for easy insertion and deletion of elements

while list supports the `for(auto x: list)` syntax we might want to add or remove elements at a specific place
```c++
int get_number(const string& s)
{
  //.begin() function returns iterator, we use ++ to move through elements, and check against .end() to stop iterating
  for (auto p = phone_book.begin(); p!=phone_book.end(); ++p)
    if (p−>name==s)
      return p−>number;

  return 0; // use 0 to represent "number not found"
}

//Adding and removing elements
void f(const Entry& ee, list<Entr y>::iterator p, list<Entry>::iterator q)
{
  phone_book.insert(p,ee); // add ee before the element referred to by p
  phone_book.erase(q); // remove the element referred to by q
}
```

### map
Container of pairs, optimized for lookup

```c++
map<string,int> phone_book {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Bertrand Arthur William Russell",345678}
};
```

Can be accessed several ways
```c++
  phone_book[some_string];
  // If some_string exists as key then value is returned, else a new key pair is created with a default value
  // The default value is dependent on type

  phone_book.find()
  phone_book.insert()
  // These functions can avoid the unwanted side effects of []
```

### unordered_map
A hash table based map for finding things like strings, functionally similar to map

## Algorithms
```c++
//Sort vector and add unique elems to list
bool operator<(const Entry& x, const Entry& y) // less than
{
  return x.name<y.name; // order Entrys by their names
}

void f(vector<Entry>& vec, list<Entry>& lst)
{
  sort(vec.begin(),vec.end()); // use < for order
  unique_copy(vec.begin(),vec.end(),lst.begin()); // don’t copy adjacent equal elements
}

//We can also create the container from scratch
list<Entry> f(vector<Entry>& vec)
{
  list<Entry> res;
  sort(vec.begin(),vec.end());
  unique_copy(vec.begin(),vec.end(),back_inserter(res)); // append to res
  return res;
}
```

`back_inserter` is a special kind of object

### Using Iterators
Collections aren't the only things that give iterators
```c++
bool has_c(const string& s, char c) // does s contain the character c?
{
  auto p = find(s.begin(),s.end(),c); //p is another iterator
  if (p!=s.end())
    return true;
  else
    return false;
}

//Short version
bool has_c(const string& s, char c) // does s contain the character c?
{
  return find(s.begin(),s.end(),c)!=s.end();
}
```

Iterators can also be put to much more complex use
```c++
vector<string::iterator> find_all(string& s, char c) // find all occurrences of c in s
{
vector<string::iterator> res;
  for (auto p = s.begin(); p!=s.end(); ++p)
    if (∗p==c)
      res.push_back(p);
  return res;
}

// This example can be templated to work with most containers
template<typename T>
using Iterator<T> = typename T::iterator;

template<typename C, typename V>
vector<Iterator<C>> find_all(C& c, V v) // find all occurrences of v in c
{
  vector<Iterator<C>> res;
  for (auto p = c.begin(); p!=c.end(); ++p)
    if (∗p==v)
      res.push_back(p);
  return res;
}

//Could be used like
string m {"Mary had a little lamb"};
for (auto p : find_all(m,'a')) // p is a str ing::iterator
  if (∗p!='a')
    cerr << "string bug!\n";
```

### Iterator Types
Iterators may be defined differently, but they all behave the same

`++` gets the iterator for the next object

`*` yields the element the iterator references

### Stream Iterators
We can handle IO with simple iterators
```c++
ostream_iterator<string> oo {cout}; //Iterator for cout

*oo = "Hello, "; //Use * to write to cout
++oo; //Actually write value
*oo = "world!\n"; // Like we used another cout << "word!\n"
```

We can perform similar tasks with cin
```c++
istream_iterator<string> ii {cin};
//We also need an ending iterator to compare against
istream_iterator<string> eos {};
```

Combining these techniques we can easily interface with the algorithms discussed above
```c++
string from, to;
cin >> from >> to; // get source and target file names

ifstream is {from}; // input stream for file "from"
istream_iterator<string> ii {is}; // input iterator for stream
istream_iterator<string> eos {}; // input sentinel

ofstream os{to}; // output stream for file "to"
ostream_iterator<string> oo {os,"\n"}; // output iterator for stream

vector<string> b {ii,eos}; // b is a vector initialized from input [ii:eos)
sort(b.begin(),b.end()); // sort the buffer

unique_copy(b.begin(),b.end(),oo); // copy buffer to output, discard replicated values

return !is.eof() || !os; // return error state
```

### Predicates
We can use predicates or lambda expressions like discussed before to perform certain tasks
```c++
void f(map<string,int>& m)
{
  auto p = find_if(m.begin(),m.end(),Greater_than{42});
  // ...
}

// or...

int cxx = count_if(m.begin(), m.end(), [](const pair<string,int>& r) { return r.second>42; });
```
