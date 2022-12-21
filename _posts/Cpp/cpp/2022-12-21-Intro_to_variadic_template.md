---
title:  "Variadic template"
excerpt: "Overveiw of std::invoke function and its capabilities"

categories:
  - cpp-tips
  - Cpp
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-12-20
last_modified_at: 2022-12-20
---

## basic usage of template

```c++
#include <iostream> 
#include <sstream>

template<typename T>
std::string to_string(const T& t)
{
    std::stringstream ss;
    ss << t;   
    return ss.str();
    // std::cout <<sd t << '\n';
}

int main()
{
    std::cout << to_string(1)<< '\n';
    to_string("hello world");
    to_string(5.9);
}
```

now let say we want to convert many things

```c++
#include <iostream> 
#include <sstream>
#include <vector>
template<typename T>
std::string to_string_impl(const T& t)
{
    std::stringstream ss;
    ss << t;   
    return ss.str();
    // std::cout <<sd t << '\n';
}
//0 case
std::vector<std::string> to_string()
{
    return {};
}
//recursive template instantiation
template<typename P1, typename ... Param> //... indicates variadic template
std::vector<std::string> to_string(const P1& p1,const Param& ...param)
{
    std::vector<std::string> s;
    s.push_back(to_string_impl(p1));

    //compiler essentially expands this to something like to_string(param1, param2, param3, ...)
    const auto remainder = to_string(param...);
    s.insert(s.end(), remainder.begin(), remainder.end());
    return s;
}

int main()
{
    const auto vec = to_string("hello", 1, 5, 3);
    for(const auto&o : vec)
    {
        std::cout << o << '\n'; 
    }
}
```

However for a such simple program, time taken to compile is quite large(measured using `/usr/bin/time/`).
Using command line tool `nm` (in linux) to dump all the symbols in a executable and pipeing that through `c++filt -t`(which demangles all of the cpp symbol names)

for more information about symble look into [Introduction to symbol visibility](https://developer.ibm.com/articles/au-aix-symbol-visibility/#:~:text=Symbol%20is%20one%20of%20the,%2Fname%2C%20and%20so%20on.) .
```bash
#full command
nm {executable name} | c++filt -t | less

#use /{function name} to find the line with corresponding symbols
```
as a result


```nasm
0000000000002ade W std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > to_string_impl<char [6]>(char const (&) [6])
0000000000002fbb W std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > to_string_impl<int>(int const&)
000000000000258c unsigned short __static_initialization_and_destruction_0(int, int)
0000000000002429 T to_string[abi:cxx11]()
000000000000276a W std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > to_string<char [6], int, int, int>(char const (&) [6], int const&, int const&, int const&)
00000000000037a9 W std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > to_string<int>(int const&)
0000000000003083 W std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > to_string<int, int>(int const&, int const&)
0000000000002bdb W std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > to_string<int, int, int>(int const&, int const&, int const&)
00000000000033b0 W __gnu_cxx::new_allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::deallocate(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*, unsigned long)
```

Here is every instantiation of to_string that we've created. 


## Better solution
we don't need recursive template instantiation. Instead,

```c++
template<typename ... Param>
std::vector<std::string> to_string(const Param& ... param)
{
    //use initializer list to initialize std::vector<std::string>
    //expand all of the to_string_impl calls for each param that is passed in
    return {to_string_impl(param) ...}
}
```
as a result you see dramatic deduction of memory usage in compile time, also time taken to compile, as well as executable size. 

