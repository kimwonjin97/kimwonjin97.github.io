---
title:  "Don't use constexpr"
excerpt: "Use static constexpr instead of just constexpr"

categories:
  - Cpp
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-09-28
last_modified_at: 2022-09-28
---
# Stop Using 'constexpr'

source: [cpp_weekley]{https://www.youtube.com/watch?v=4pKtPWcl1Go}



```c++
constexpr int get_value(int value)
{
	return value*2
}

int main()
{
  int value = get_value(6); //maybe? up to the optimizer.
  // is value calculated at compile time?
  
  static_assert(value == 12); //error: non-constant condition for static assertion
  
  constexpr int value = get_value(6) //done in compile time? maybe we haven't done anything to require the compiler to calulate this at compile time.
    
  //gcc is very aggressvie with constexpr
  //llvm somewhere between agressive and lazy
  //msvc is lazy
  
  
  return value;
}
```

We don't know where `constexpr int value = get_value(6)` will be done in compile time. Onlyway to force compile time optimization is by using `static_assert`



```c++
#include <array>

constexpr std::array<int, 1000> get_value()
{
  std::array<int, 1000> retval{};
  int count = 0; 
  for(auto &val : retval)
  {
    val = count*3;
    ++count;
  }
  return retval;
}

int main()
{
  constexpr auto values = get_value();
  //constexpr values are stack values
  // it was not required to do this
  return values[879]
}

int main2()
{
  const int *p = nullptr;
  {
    constexpr auto values = get_value(); // this is from the stack
    //so p is point to temporary. When values gets pop from the stack, p is pointing to invalid object.
    //constexpr values are stack values (unless they are statics)
    p = &values[985];
  }
  //the warning is however disabled with optimizer turned on.
  return *p;
}
```



## Notes

According to cpp standard,

> **Constant initialization is performed if a variable or temporary object with static or thread storage duration is constant-initialized** (7.7). If constant initialization is not performed, a variable with static storage duration (6.7.5.1) or thread storage duration (6.7.5.2) is zero-initialized (9.4). Together, zero-initialization and constant initialization are called static initialization; all other initialization is dynamic initialization. **All static initialization strongly happens before (6.9.2.1) any dynamic initialization.** [Note: The dynamic initialization of non-local variables is described in 6.9.3.3; that of local static variables is described in 8.8. — end note]

```c++
static constexpr auto values = get_value();
```

this forces initialization at compile-time, through the means of "constant initialization"

## best practice!

1. you must run all tests with addresss sanitizer enabled (-fsantize=address) (ASAN)
2. you must run both release and debug build with ASAN, with tests.
3. you rarely want constexpr variables
4. you almost always mean static constexpr instead

