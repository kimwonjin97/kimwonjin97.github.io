---
title:  "Category of classes and struct in C++14 or above"
excerpt: "Investigate the difference between trivial, standard-layout, and POD type"

categories:
  - Cpp
  - cpp-tips
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-12-24
last_modified_at: 2022-12-24
---

More detail post at: 
https://learn.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-170


> Term:
>> Layout: how the members of an object of class, struct or union type are arranged in memory.

Starting from c++14, class and struct has been categorized into three different types, which are <span style="color:red">Trivial type</span>, <span style="color:red">Standard Layout type</span>, and <span style="color:red">POD(Plain Old Data) type</span>. Categorizing the type helped compiler, c++ program and metaprograms to reason about the suitability of any given type for operations that depend on a particular memory layout. 

Standar library has function templates which are `is_trivial<T>`, `is_standard_layout<T>`, `is_pod<T>` for identifying the class type.


## Trivial type

<span style="color:blue">Trivial types have a trivial default constructor, trivial copy constructor, trivial copy assignment operator and trivial destructor<span style="color:blue">. In each case, trivial means the constructor/operator/destructor is not user-provided and belongs to a class that has

1. no virtual functions or virtual base classes,
2. no base classes with a corresponding non-trivial constructor/operator/destructor
3. no data members of class type with a corresponding non-trivial constructor/operator/destructor

## Standard layout types

Standard-layout types can have user-defined special member functions. In addition, standard layout types have these characteristics:

- no virtual functions or virtual base classes
- all non-static data members have the same access control
- all non-static members of class type are standard-layout
- any base classes are standard-layout
- has no base classes of the same type as the first non-static data member.
- meets one of these conditions:
    - no non-static data member in the most-derived class and no more than one base class with non-static data members, or
    - has no base classes with non-static data members

## POD types
class or struct is POD type when it is <span style="color:red">both trivial and standard-layout</span>.


## Sample Example from MS
```c++
#include <type_traits>
#include <iostream>

using namespace std;

struct B
{
protected:
   virtual void Foo() {}
};

// Neither trivial nor standard-layout
struct A : B
{
   int a;
   int b;
   void Foo() override {} // Virtual function
};

// Trivial but not standard-layout
struct C
{
   int a;
private:
   int b;   // Different access control
};

// Standard-layout but not trivial
struct D
{
   int a;
   int b;
   D() {} //User-defined constructor
};

struct POD
{
   int a;
   int b;
};
```

## Literal Type(from MS)
Whose layout can be determined at compiled type
    1. void
    2. scalar types
    3. references
    4. Arrays of void, scalar types or reference
    5. A class that has a trivial destructor, and one or more constexpr constructors that are not move or copy constructors. Additionally, all its non-static data members and base classes must be literal types and not volatile. 