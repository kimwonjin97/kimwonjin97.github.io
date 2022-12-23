---
title:  "Inheriting from Lambda"
excerpt: "Explore different ways to inherit from Lambda"

categories:
  - cpp-tips
  - Cpp
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-12-23
last_modified_at: 2022-12-23
---
## Inheriting from Lambda

More info in regards to lambda and std::decay could be found here: [lambda](https://en.cppreference.com/w/cpp/language/lambda), [decay](https://en.cppreference.com/w/cpp/types/decay). In this post we will 

```c++
#include <functional>
#include <type_traits>

template<typename L1, typename L2>
struct S : L1, L2
{
    S(L1 l1, L2 l2): L1(std::move(l1)), L2(std::move(l2)) {}
    //Since lambda is standard type
    //We want to dispatch to appropriate operator
    using L1::operator();
    using L2::operator();
};

template<typename L1, typename L2>
auto make_combined(L1 &&l1, L2 &&l2)
{
    //type of L1 and L2 could be rvalue or other type
    return S<std::decay_t<L1>, std::decay_t<L2>>(std::forward<L1>(l1), std::forward<L2>(l2));
}

auto main() -> int
{
    auto l= [](){return 4;};
    auto l2= [](int i){return i * 10;};

    auto combined = make_combined(l , l2);
    return combined(10);
}
``` 


We could utilize c++17 <span style="color:red">class template automatic type deduction</span>, and elminate `make_combined helper`. Then the result looks like:

```c++
#include <functional>
#include <type_traits>

template<typename L1, typename L2>
struct S : L1, L2
{
    S(L1 l1, L2 l2): L1(std::move(l1)), L2(std::move(l2)) {}
    //Since lambda is standard type
    //We want to dispatch to appropriate operator
    using L1::operator();
    using L2::operator();
};

auto main() -> int
{
    auto l= [](){return 4;};
    auto l2= [](int i){return i * 10;};

    auto combined = S(l , l2); 
    return combined(10);
}
```


## Using variadic template
The problem with the code above is that we are restricted to inheriting from two lambda function. If we want to add more lambda function then we need to add `using LX::operator()` individual which is cumbersome. To Resolve this issue we could use variadic template

```c++
#include <memory>
#include <utility>
//using only this struct cause problem since
//constructor has different template parameter pack than our struct does.
//hence c++17 type deduction doesn't work
template<typename ...B>
struct Merged : B...
{
    template<typename ...T>
    Merged(T && ...t) : B(std::forward<T>(t)) ...
    {
    }

    using B::operator() ...;
};

//we could fix the problem above with deduction guide.
template<typename ...T>
Merged(T...)->Merged<std::decay_t<T>...>;

auto main() -> int
{
    auto l= [](){return 4;};
    auto l2= [](int i){return i * 10;};

    Merged merged(l , l2, [](const double d) {return d * 3.2;}
                , [i=std::make_unique<int>(5)](char){}); // use could also pass in non copyable lambda that has unique pointer in its capture list
    return merged(10);
}

```