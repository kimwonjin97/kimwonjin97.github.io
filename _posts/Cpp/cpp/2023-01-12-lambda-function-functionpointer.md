---
title:  "C++ Lambda, std::function, function pointer"
excerpt: "Explore different use cases for C++ Lambda, std::function, and function pointer"

categories:
  - Cpp
  - cpp-tips
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2023-01-12
last_modified_at: 2023-01-12
---

>C++ lambda is **not** std::function

```c++
#include <functional>
int main()
{
    // CASE 1
    // lambda: a C++ language construct for defining an anonymous function
    auto add = [](const int x, const int y)
    {
        return x + y;
    };
    return add(4, 5);

    // CASE2
    // std::function: a type-erased wrapper around a "callable"
    // std::function<int (const int, const int)> func; // holder/abstraction of a function.
    std::function<int (const int, const int)> func{add};
    return func(4,5);
}

```


```c++
int add(int x, int y)
{
    return x + y;
}

int mul(int x, int y)
{
    return x * y;
}

int sub(int x, int y)
{
    return x - y;
}

//////////////////

int call(int (*f)(int, int))
{
    return f(2, 3);
}

//this has much more line  in assembly due to all the abstraction happening 
int call(const std::function<int (int, int)> &f)
{
    return f(2, 3);
}
//////////////////

int main()
{


    int val1 = 15;
    std::vector<std::function<int (int, int)>> operations;
    operations.emplace_back([](int x, int y) { return x + y; });
    operations.emplace_back([](int x, int y) { return x * y; });
    operations.emplace_back([](int x, int y) { return x - y; });
    operations.emplace_back([=](int x, int y) { return val1 + x - y; }); //works but not for cases below


    std::vector<int (*)(int, int)> operations;
    operations.emplace_back([](int x, int y) { return x + y; }); //this is possible for "captureless" lambdas
    operations.emplace_back([](int x, int y) { return x * y; });
    operations.emplace_back(add);

}

```
Anything that can be called with a specified set of parameters can be used in the place of std::function