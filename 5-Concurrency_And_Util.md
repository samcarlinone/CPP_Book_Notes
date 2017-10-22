# A Tour of C++: Concurrency and Utilities
_(The C++ Programming Language Chapter 5, ed. 4, by Bjarne Stroustrup)_

## Resource Management
In C++ correctly combining constructor / destructor is crucial to avoid memory leaks

### unique_ptr and shared_ptr
These two STL pointers help to manage objects allocated from the free store

```c++
void f(int i, int j)
{
  X* p = new X;
  unique_ptr<X> sp {new X};

  if(i < 99) throw Z{}; //Throws an error
  if(j < 77) return; //Returns early

  delete p; //If anything happens to prevent this being called memory is leaked
}
```

`shared_ptr` works similarly to `unique_ptr`, but it is copied rather than moved, meaning `shared_ptr` can be passed to multiple functions

it should also be noted that these pointers simply avoid resource leaks, and do not handle data races or other shenanigans

## Concurrency
### Tasks and thread(s)
Simple example of threads, found in `<thread>`
```c++
void f(); // function

struct F { // function object
  void operator()(); // F’s call operator
};

void user()
{
  thread t1 {f}; // f() executes in separate thread
  thread t2 {F()}; // F()() executes in separate thread

  t1.join(); // wait for t1
  t2.join(); // wait for t2
}
```

unlike processes, threads can share data, which can lead to problems if done incorrectly

### Passing Arguments
```c++
void f(vector<double>& v); // function do something with v

struct F { // function object: do something with v
  vector<double>& v;
  F(vector<double>& vv) :v{vv} { }
  void operator()(); // application operator
};

int main()
{
  vector<double> some_vec {1,2,3,4,5,6,7,8,9};
  vector<double> vec2 {10,11,12,13,14};

  thread t1 {f,some_vec}; // f(some_vec) executes in a separate thread
  thread t2 {F{vec2}}; // F(vec2)() executes in a separate thread

  t1.join();
  t2.join();
}
```

Thread is an example of variadic templates in action, as that is how it receives arguments

### Receiving Arguments
We can pass data by non-`const` ref, and there are other ways, but this one works

### Sharing Data
To prevent accidentaly simultaneous manipulation of data locks are used

Implemented as a `mutex`, which threads access by using `lock()`
```c++
mutex m; // controlling mutex
int sh; // shared data

void f()
{
  unique_lock<mutex> lck {m}; // acquire mutex, calls lock() automatically
  sh += 7; // manipulate shared data
} // mutex released when unique_lock goes out of scope
```

Often to reduce confusion about which `mutex`es belong to which object they are integrated directly
```c++
class Record {
public:
  mutex rm;
  // ...
};
```

While `mutex`es are very helpful trying to `lock()` multiple resources can cause deadlocks

Imagine if two threads both wanted to lock the same two resources, if each locked one, they could end up deadlocked

However `lock` and `defer_lock` handle the details to prevent this from happening
```c++
void f()
{
  // ...
  unique_lock<mutex> lck1 {m1,defer_lock}; // defer_lock: don’t yet try to acquire the mutex
  unique_lock<mutex> lck2 {m2,defer_lock};
  unique_lock<mutex> lck3 {m3,defer_lock};
  // ...
  lock(lck1,lck2,lck3); // acquire all three locks
  // ... manipulate shared data ...
} // implicitly release all mutexes
```

### Waiting for Events
```c++
using namespace std::chrono;
auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();
cout << duration_cast<nanoseconds>(t1−t0).count() << " nanoseconds passed\n";
```

`this_thread` refers to the running thread, or in this case the main thread

`duration_cast` allows easy time manipulation, found in `<chrono>`

`condition_variable` provides a relatively simple way of notifying waiting threads

```c++
class Message { // object to be communicated
  // ...
};

queue<Message> mqueue; // the queue of messages
condition_variable mcond; // the variable communicating events
mutex mmutex; // the locking mechanism

void consumer()
{
  while(true) {
    unique_lock<mutex> lck{mmutex};           // acquire mmutex
    while (mcond.wait(lck)); // release lck and wait;
                                              // re-acquire lck upon wakeup

    auto m = mqueue.front(); // get the message
    mqueue.pop();
    lck.unlock(); // release lck
    // ... process m ...
  }
}

void producer()
{
  while(true) {
    Message m;
    // ... fill the message ...
    unique_lock<mutex> lck {mmutex}; // protect operations
    mqueue.push(m);
    mcond.notify_one(); // notify
  } // release lock (at end of scope)
}
```

### Communicating Tasks
all these use `<future>`

#### future and promise
```c++
future<X> fx;
X v = fx.get(); // if necessary, wait for the value to get computed

//Somewhere else
void f(promise<X>& px) // a task: place the result in px
  {
  // ...
  try {
    X res;
    // ... compute a value for res ...
    px.set_value(res);
  }
  catch (...) { // oops: couldn’t compute res
    // pass the exception to the future’s thread:
    px.set_exception(current_exception());
  }
}
```

for `future` we use `get()`, for `promise` we use `set_value()` and `set_exception()`

#### packaged_task
```c++
double accum(double∗ beg, double ∗ end, double init)
{
  // compute the sum of [beg:end) starting with the initial value init
  return accumulate(beg,end,init);
}

double comp2(vector<double>& v)
{
  using Task_type = double(double∗,double∗,double); // type of task

  packaged_task<Task_type> pt0 {accum}; // package the task (i.e., accum)
  packaged_task<Task_type> pt1 {accum};

  future<double> f0 {pt0.get_future()}; // get hold of pt0’s future
  future<double> f1 {pt1.get_future()}; // get hold of pt1’s future

  double∗ first = &v[0];
  thread t1 {move(pt0),first,first+v.size()/2,0}; // star t a thread for pt0
  thread t2 {move(pt1),first+v.size()/2,first+v.size(),0}; // star t a thread for pt1

  // ...

  return f0.get()+f1.get(); // get the results
}
```

note that `packaged_task` cannot be copied, only moved

#### async()
```c++
double comp4(vector<double>& v)
// spawn many tasks if v is large enough
{
  if (v.size()<10000) return accum(v.begin(),v.end(),0.0);

  auto v0 = &v[0];
  auto sz = v.size();

  auto f0 = async(accum,v0,v0+sz/4,0.0); // first quarter
  auto f1 = async(accum,v0+sz/4,v0+sz/2,0.0); // second quarter
  auto f2 = async(accum,v0+sz/2,v0+sz∗3/4,0.0); // third quarter
  auto f3 = async(accum,v0+sz∗3/4,v0+sz,0.0); // fourth quarter

  return f0.get()+f1.get()+f2.get()+f3.get(); // collect and combine the results
}
```

note that `async()` will use as many or as few threads as it decides, this means it should only be used
when we specifically don't want to deal with multithreading

## Standard Utility Components
### Time
```c++
using namespace std::chrono;

auto t0 = high_resolution_clock::now();
do_work();
auto t1 = high_resolution_clock::now();
cout << duration_cast<milliseconds>(t1-t0).count() << "msec\n";
```

high_resolution_clock::now() returns a `time_point` object, subtracting two `time_point`s gives a `duration`

__Different clocks return different units, using `duration_cast` is highly recommended__

### Type Function
```c++
constexpr float min = numeric_limits<float>::min(); //smallest positive float
constexpr int szi = sizeof(int); // the number of bytes in an int
```

#### iterator_traits
```c++
template<typename Ran> // for random-access iterators
void sort_helper(Ran beg, Ran end, random_access_iterator_tag) // we can subscript into [beg:end)
{
  sort(beg,end); // just sort it
}

template<typename For> // for forward iterators
void sort_helper(For beg, For end, forward_iterator_tag) // we can traverse [beg:end)
{
  vector<decltype(∗beg)> v {beg,end}; // initialize a vector from [beg:end)
  sort(v.begin(),v.end());
  copy(v.begin(),v.end(),beg); // copy the elements back
}

template<typename C>
using Iterator_type = typename C::iterator; // C’s iterator type

template<typename Iter>
using Iterator_category = typename std::iterator_traits<Iter>::iterator_category; // Iter’s category

template<typname C>
void sort(C& c)
{
  using Iter = Iterator_type<C>;
  sort_helper(c.begin(),c.end(), Iterator_category<Iter>{});
}
```

this code uses templating, `delctype()`, and `iterator_traits` from `<iterator>` to work

this style is called _tag dispatch_ and is used throughout the standard library

#### Type Predicates
```c++
bool b1 = Is_arithmetic<int>(); // yes, int is an arithmetic type
bool b2 = Is_arithmetic<string>(); // no, std::string is not an arithmetic type
```

these predicates are found in `<type_traits>`, there are many for checking type, class, base, etc.

```c++
template<typename Scalar>
class complex {
  Scalar re, im;
public:
  static_assert(Is_arithmetic<Scalar>(), "Sorry, I only support complex of arithmetic types");
  // ...
};

//uses defined type function

template<typename T>
constexpr bool Is_arithmetic()
{
  return std::is_arithmetic<T>::value ;
}
```

### pair and tuple
a pair supports two elements, a tuple supports many
```c++
pair<int, string> bob(some_num, some_string);
// or
auto bob = make_pair(some_num, some_string);

//accessed with first and second
if(bob.first == 10)
  cout << bob.second;
```

```c++
tuple<string, int, double> alice("ja", 1, 42.1);

auto alice = make_tuple(string("ja"), 1, 42.1);

if(get<0>(alice) == "ja")
  cout << get<1>(alice);
```

## Regular Expressions
Regex library under `<regex>` header.
```c++
regex pat (R"(\w{2}\s*\d{5}(−\d{4})\?)"); //ZIP code pattern: XXddddd-dddd and variants, \? shouldn't be escaped
cout << "pattern: " << pat << '\n';
```

`R"()"` is a _raw string literal_, meaning we can use backslashes and quotes without escaping

```c++
int lineno = 0;
for (string line; getline(cin,line);) { // read into line buffer
  ++lineno;
  smatch matches; // matched strings go here
    if (regex_search(line ,matches,pat)) // search for pat in line
      cout << lineno << ": " << matches[0] << '\n';
}
```

`matches` var is of type `smatch`, short for submatch, is a vector

`matches[0]` is the complete match

## Math
### Mathematical Functions and Algorithms
`<cmath>` holds normal number functions, like `sqrt()`, `log()`, and `sin()`

Complex versions of these functions are found in `<complex>`

`<numeric>` holds generalized numeric algorithms like `accumulate()`
```c++
void f()
{
  list<double> lst {1, 2, 3, 4, 5, 9999.99999};
  auto s = accumulate(lst.begin(),lst.end(),0.0); // calculate the sum
  cout << s << '\n'; // print 10014.9999
}
```

### Complex Numbers
`complex` is templated
```c++
template<typename Scalar>
class complex {
public:
  complex(const Scalar& re ={}, const Scalar& im ={});
  // ...
};

void f(complex<float> fl, complex<double> db)
{
  complex<long double> ld {fl+sqrt(db)};
  db += fl∗3;
  fl = pow(1/fl,2);
  // ...
}
```

### Random Numbers
`<random>` provides many types of distribution that hook into the random _engine_ that produces values

`uniform_int_distribution` (all ints produced equally likely), `normal_distribution` (bell curve), `exponential_distribution` (exponential growth)

```c++
using my_engine = default_random_engine; // type of engine
using my_distribution = uniform_int_distribution<>; // type of distribution

my_engine re {}; // the default engine
my_distribution one_to_six {1,6}; // distribution that maps to the ints 1..6
auto die = bind(one_to_six,re); // make a generator

int x = die(); // roll the die: x becomes a value in [1:6]
```

`bind()` is a STL function that creates a function object that invokes the first argument when called, the second argument provided will be the argument provided

Here is a simple random int class
```c++
class Rand_int {
public:
  Rand_int(int low, int high) :dist{low,high} { }
  int operator()() { return dist(re); } // draw an int
private:
  default_random_engine re;
  uniform_int_distribution<> dist;
};

int main()
{
  Rand_int rnd {0,4}; // make a unifor m random number generator

  vector<int> histogram(5); // make a vector of size 5
  for (int i=0; i!=200; ++i)
    ++histogram[rnd()]; // fill histogram with the frequencies of numbers [0:4]

  for (int i = 0; i!=mn.size(); ++i) { // write out a bar graph
    cout << i << '\t';
    for (int j=0; j!=mn[i]; ++j) cout << '∗';
    cout << endl;
  }
}
```

### Vector Arithmetic
`vector` isn't optimized for mathematical vector operations, `<valarray>` is optimized for maths, although it also more limited
```c++
void f(valarray<double>& a1, valarray<double>& a2)
{
  valarray<double> a = a1∗3.14+a2/a1; // numeric array operators \*, +, /, and =
  a2 += a1∗3.14;
  a = abs(a);
  double d = a2[7];
  // ...
}
```

### Numeric Limits
`<limits>` the STL header helps examine built-in types
```c++
static_assert(numeric_limits<char>::is_signed,"unsigned characters!");
static_assert(100000<numeric_limits<int>::max(),"small ints!");
```
