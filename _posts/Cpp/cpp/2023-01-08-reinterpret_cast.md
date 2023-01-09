---
title:  "Stop using reinterpret_cast"
excerpt: "reinterpret_cast"

categories:
  - Cpp
  - cpp-tips
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2023-01-08
last_modified_at: 2023-01-08
---

There are very few ways to use reinterpret_cast without invoking undefined behavior


```c++
#include <cstring>
struct S
{
    int i;
    int j;
};

// we are trying to access an object whose life time never began
void update_data(char* blob, int new_value)
{
    reinterpret_cast<S*>(blob)[2].j = new_value; //mov   dword ptr [rdi + 20], esi
}

void update_date_not_undefined(char* blob, int new_value)
{
    S obj[3];
    memcpy(obj, blob, sizeof(obj));
    obj[2].j = new_value;
    memcpy(blob, obj, sizeof(obj));

    //std::bitcast<> : cpp20 
}
```