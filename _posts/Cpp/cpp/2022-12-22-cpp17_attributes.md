---
title:  "some attributes in c++17"
excerpt: "Explore usage of [[fallthrough]], [[maybe_unused]], and [[nodiscard]]"

categories:
  - cpp-tips
  - Cpp
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-12-22
last_modified_at: 2022-12-22
---


# [[fallthrough]]
When we use `[[fallthrough]]` within the switch statement, we tell compiler that we intended to fall through from one case to another. 
```c++
void do something(){}

void do_something_else(){}

int main(int argc, const char*[])
{
    switch(argc)
    {
        case 1:
            do_something;
            [[fallthrough]];
        case 2:
            do_something_else;
    }
}

```

# [[maybe_unused]]

Usful for situation when compiler is going to warn you that something has unused that was defined.

```c++
#include <cassert>

[[maybe_unused]] void something()

int main(int /*argc*/, [[maybe_unused]] const char *argv[])
{
  [[maybe_unused]] int i=6;
  assert(i == 6);
}
```

# [[no_discard]]

Tells compiler that return value of the function is important and should not be discarded.

```c++
[[nodiscard]]int something()
{
  return 1;
}

int main()
{
  return something;
}
```

`[[nodiscard]]` could also be applied to struct, enum, class, union, etc.

```c++
struct [[nodiscard]] MyError
{  
};

enum [[nodiscard]] MyError
{
  VAL1,
  VAL2
};
```

when trying to avoid exception while getting some level of guarentee that we are appropriately handling error code. 