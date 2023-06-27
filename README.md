# cppdeductionguides
Info about deduction guides in modern cpp

# Type deduction

**[ Source: https://pvs-studio.com/en/blog/posts/cpp/1006/ ]**

In C++ type deduction refers to the automatic or half-automatic detection of the type of a variable. In this case, the compiler is able to deduce the type through the initializer (of the assigned expression).

The type-deduction mechanism determines only the way in which the type is deduced — as a reference or as a non-reference type. So, we still end up with one of the options from the first paragraph as a result of the type deduction: either passing by reference, or passing by value. We're going to briefly discuss the ways of the type deduction to get the full picture.

# 1. auto

Since C++11, we're able to use the auto keyword to deduce the type of a variable through its initializer. If we use the auto keyword without any additional qualifiers and & (as is) during the type deduction of the variable type, then a reference declaration and the const and volatile type qualifiers are ignored. In this case storing by value occurs. Check the type deduction:

```cpp
void main()
{
  // Output: Xray ctor, value is 1
  auto xray = Xray{"1"}; // type = Xray
} // Output: Xray dtor, value is 1
```

 The examples of the qualifiers dropped during the auto type deduction:

```cpp
int i = 0;
int& iRef = i;
const int& iConstRef = i;
volatile int& iVolatileRef = i;
const volatile int& iCVRef = i;
int* iPtr = &i;

auto _1 = i;            // type = int
auto _2 = iRef;         // type = int
auto _3 = iConstRef;    // type = int
auto _4 = iVolatileRef; // type = int
auto _5 = iCVRef;       // type = int
auto _6 = iPtr;         // type = int*
```

To add qualifiers and a reference declaration (or to save them during the type deduction), you need to specify them next to the auto keyword. When qualifiers and reference types are added, the lifetime is extended through a const lvalue reference (cppinsights):

```cpp
void main()
{
  // Output: Xray ctor, value is 1
  const auto& xray = Xray{"1"}; // type = const Xray&
} // Output: Xray dtor, value is 1
```

You can also specify auto&& so the type deduction mechanism would be quite similar to perfect forwarding. When you use auto&&, the lifetime is extended through an rvalue reference :

```cpp
void main()
{
  // Output: Xray ctor, value is 1
  auto&& xray = Xray{"1"}; // type = Xray&&
} // Output: Xray dtor, value is 1
```

**auto&&** covers only two cases. When returning lvalues, the return type would be T& (lvalue-reference); for rvalues, including xvalues and prvalues, the return type would be T&& (rvalue-reference). 

Learn more about perfect forwarding in the 4. template paragraph.

 The examples of the auto&& type deduction:

```cpp
int i = 0;
const int& iConstRef = i;

auto&& _ = i;           // type = int&
auto&& _1 = iConstRef;  // type = const int&
auto&& _2 = 4;          // type = int&&
```


# 2. decltype

Since C++11 with the introduction of the decltype keyword, we're able to obtain the type of an expression at compile time. The examples of the type deduction with the decltype keyword:

```cpp
int i = 0;
const int& iConstRef = i;
int&& iRvalueRef = 1;

decltype(i) _1 = i;                              // type = int
decltype(iConstRef ) _2 = iConstRef;             // type = const int&
decltype(iRvalueRef) _3 = std::move(iRvalueRef); // type = int&&
decltype(3) _4 = 3;                              // type = int
```

So, if you use the decltype keyword, then storing by value occurs:

```cpp
void main()
{
  // Output: Xray ctor, value is 1
  decltype(Xray{"1"}) xray = Xray{"1"}; // type = Xray
} // Output: Xray dtor, value is 1
```

If you need to deduce the type of an object that does not have the required constructor (in particular, the default constructor), then you can use std::declval:

```cpp
decltype(std::declval<Xray>()) xray = Xray{"1"}; // type = Xray&&
```

The type deduction at compile time means that an expression passed to decltype is processed not during program execution, but during compilation. The compiler only checks which type is the result of the expression and then substitutes it.

Example: in the decltype(2+2) expression, the result of the addition (2+2) is not processed, since the compiler sees this expression only in terms of types: (int)+(int), so the result is int.

You may have noticed that using decltype in this way is not very convenient:

1. You get a lot of extra information that completely or partially duplicates the expression being assigned.

2. Once in a while you have to use various workarounds (such as std::declval) to deduce the type.

That's why decltype was modified into decltype(auto) in the next standard.


# 3. decltype(auto)


Since C++14, we're able to pass the auto keyword to decltype as a parameter. With decltype(auto) we can deduce the same type as the one of the expressions assigned, thus the qualifiers and a reference declaration are saved.

The examples of the type deduction with decltype(auto):

```cpp
int i = 2;
const int& iConstRef = 0;

decltype(auto) _1 = 1;            // type = int
decltype(auto) _2 = iConstRef;    // type = const int&
decltype(auto) _3 = std::move(i); // type = int&&
```
**decltype(auto)** covers three cases. When returning lvalues, the return type would be T& (lvalue-reference); for xvalues, the return type would be T&& (rvalue-reference); for prvalues, the return type would be T (non-reference, i.e. return by-value).

Thus, if you use decltype(auto), then storing by value occurs.

```cpp
void main()
{
  // Output: Xray ctor, value is 1
  decltype(auto) xray = Xray{"1"}; // type = Xray
} // Output: Xray dtor, value is 1
```

# 4. template

The template type deduction is pretty similar to the auto type deduction.

If we specify the template type T (from template<typename T>) without any additional qualifiers and & (as is), during the template argument deduction in a function template, then a reference declaration and the const and volatile qualifiers are ignored. In this case storing by value occurs:

```cpp
template<typename T> 
void foo(T param)
{}

void main()
{
  // Output: Xray ctor, value is 1
  foo(Xray{"1"}); // type = Xray
} // Output: Xray dtor, value is 1
```

The examples of the qualifiers dropped during the template type deduction:

```cpp
template<typename T> 
void foo(T param)
{}

int i = 0;
int& iRef = i;
const int& iConstRef = i;
volatile int& iVolatileRef = i;
const volatile int& iCVRef = i;
int* iPtr = &i;

foo(i);            // type = int
foo(iRef);         // type = int
foo(iConstRef);    // type = int
foo(iVolatileRef); // type = int
foo(iCVRef);       // type = int
foo(iPtr);         // type = int*
```

To add qualifiers and a reference declaration (or to save them during the type deduction), you need to specify them next to the name of a template parameter.

In this case, the lifetime is extended via a const lvalue reference:

```cpp
template<typename T> 
void foo(const T& param)
{}

void main()
{
  // Output: Xray ctor, value is 1
  foo(Xray{"1"}); // type = const Xray&
} // Output: Xray dtor, value is 1
```

You can also specify T&& (forwarding reference) so the type deduction and the value passing mechanisms would be quite similar to perfect forwarding.

In this case, the lifetime is extended via an rvalue reference::

```cpp
template<typename T> 
void foo(T&& param)
{}
void main()
{
  // Output: Xray ctor, value is 1
  foo(Xray{"1"}); // type = Xray&&
} // Output: Xray dtor, value is 1
```

The examples of the T&& type deduction:

```cpp
template<typename T> 
void foo(T&& param)
{}

int i = 0;
const int& iConstRef = i;

foo(i);            // type = int&
foo(4);            // type = int&&
foo(std::move(i)); // type = int&&
foo(iConstRef);    // type = const int&
```
