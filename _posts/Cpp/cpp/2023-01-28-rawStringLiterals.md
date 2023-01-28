---
title:  "Raw string literals"
excerpt: "Some syntax of raw string literal"

categories:
  - Cpp
  - cpp-tips
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2023-01-28
last_modified_at: 2023-01-28
---


```c++
#include <iostream>

int main()
{
    std::cout << "Hello World";

    //when we want to print single string that is in multiple line:

    std::cout << "Hello"

    "World"; //adjacent string are joined together by the c++ during pre-processing

    //alternatively:

    std::cout << R"(Hello" 

    "World)"; //white space are also printed out


}

```

Without the help of raw string printing something like path would be cumbersome. 

```c++
std::cout << "c:\\program File\\World";
```

However what do we do if we want to print something like " in the middle of the path? 

```c++
std::cout << R"(C:\program)" Files\World)"; //error

//correct version

std::cout << R"stuff(c:\Program)" Files\World)stuff"; // this raw string literal allow us to make our own terminating tokens on either end of our string ("stuff" could be anything)
``` 


