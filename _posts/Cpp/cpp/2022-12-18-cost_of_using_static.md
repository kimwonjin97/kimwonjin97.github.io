---
title:  "Cost of Using Statics"
excerpt: "quantifying the cost of accessing static variable"

categories:
  - cpp-tips
  - Cpp
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2022-12-18
last_modified_at: 2022-12-18
---

In C++11, it guarantees that the initialization of static variables happens in a thread safe fashion. Lets look at the code below

```c++
#include <string>

struct C {
    static const std::string &magic_static()
    {
        static const std::string s = "bob";
        return s;
    }
    const std::string &s = magic_static();

    const std::string &magic_static_ref()
    {
        return s;
    }
};

auto main() -> int
{
    C::magic_static().size();
    C::magic_static().size();
    C::magic_static().size();
    return C::magic_static().size();
}
```
As you could see from the code we first initialized the string reference variable s with the return value of the magic_static static function when the struct C is initialized. So the question we want to ask is <span style="color:red">how much overhead there is in accessing a static variable.</span> Considering the fact that static variable  is guaranteed to be constructed in a thread safe manner requires that the compiler must have some way of determining if the variable has already been constructed and if not, performing  a lock and constructing the variable for us. Let's  see how this translates to assembly code. (x86-64 gcc 12.2 used, with -std=c++14 -Wall -O2)

```nasm 
C::magic_static[abi:cxx11]():
        movzx   eax, BYTE PTR guard variable for C::magic_static[abi:cxx11]()::s[rip]
        test    al, al
        je      .L13
        mov     eax, OFFSET FLAT:C::magic_static[abi:cxx11]()::s
        ret
.L13:
        sub     rsp, 8
        mov     edi, OFFSET FLAT:guard variable for C::magic_static[abi:cxx11]()::s
        call    __cxa_guard_acquire
        test    eax, eax
        jne     .L14
        mov     eax, OFFSET FLAT:C::magic_static[abi:cxx11]()::s
        add     rsp, 8
        ret
.L14:
        mov     eax, 28514
        mov     edx, OFFSET FLAT:__dso_handle
        mov     esi, OFFSET FLAT:C::magic_static[abi:cxx11]()::s
        mov     edi, OFFSET FLAT:_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev
        mov     QWORD PTR C::magic_static[abi:cxx11]()::s[rip], OFFSET FLAT:C::magic_static[abi:cxx11]()::s+16
        mov     WORD PTR C::magic_static[abi:cxx11]()::s[rip+16], ax
        mov     BYTE PTR C::magic_static[abi:cxx11]()::s[rip+18], 98
        mov     QWORD PTR C::magic_static[abi:cxx11]()::s[rip+8], 3
        mov     BYTE PTR C::magic_static[abi:cxx11]()::s[rip+19], 0
        call    __cxa_atexit
        mov     edi, OFFSET FLAT:guard variable for C::magic_static[abi:cxx11]()::s
        call    __cxa_guard_release
        mov     eax, OFFSET FLAT:C::magic_static[abi:cxx11]()::s
        add     rsp, 8
        ret
main:
        sub     rsp, 8
        call    C::magic_static[abi:cxx11]()
        call    C::magic_static[abi:cxx11]()
        call    C::magic_static[abi:cxx11]()
        call    C::magic_static[abi:cxx11]()
        mov     eax, DWORD PTR [rax+8]
        add     rsp, 8
        ret
guard variable for C::magic_static[abi:cxx11]()::s:
        .zero   8
C::magic_static[abi:cxx11]()::s:
        .zero   32

```

As you could see `C::magic_static[abi:cxx11]()` is called 4 times which checks if values have been initialized, if not been initialized jumps to the initialization routine if has been initialized simply return the reference to the contained value. As a result `eax, BYTE PTR guard variable for C::magic_static[abi:cxx11]()::s[rip]` responsible for checking initialization  would at least be called 4 times to insure thread safety. 


Now lets modify the code
```c++
#include <string>

struct C {
    static const std::string &magic_static()
    {
        static const std::string s = "bob";
        return s;
    }
    const std::string &s = magic_static();

    const std::string &magic_static_ref()
    {
        return s;
    }
};

auto main() -> int
{
    C c;
    c.magic_static_ref().size();
    c.magic_static_ref().size();
    c.magic_static_ref().size();
    return c.magic_static_ref().size();
}
```
the corresponding assembly code is following

```nasm
main:
        movzx   eax, BYTE PTR guard variable for C::magic_static[abi:cxx11]()::s[rip]
        test    al, al
        je      .L13
        mov     eax, DWORD PTR C::magic_static[abi:cxx11]()::s[rip+8]
        ret
.L13:
        push    rcx
        mov     edi, OFFSET FLAT:guard variable for C::magic_static[abi:cxx11]()::s
        call    __cxa_guard_acquire
        test    eax, eax
        jne     .L14
.L3:
        mov     eax, DWORD PTR C::magic_static[abi:cxx11]()::s[rip+8]
        pop     rdx
        ret
.L14:
        mov     edx, OFFSET FLAT:__dso_handle
        mov     esi, OFFSET FLAT:C::magic_static[abi:cxx11]()::s
        mov     edi, OFFSET FLAT:_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev
        mov     QWORD PTR C::magic_static[abi:cxx11]()::s[rip], OFFSET FLAT:C::magic_static[abi:cxx11]()::s+16
        mov     WORD PTR C::magic_static[abi:cxx11]()::s[rip+16], 28514
        mov     BYTE PTR C::magic_static[abi:cxx11]()::s[rip+18], 98
        mov     QWORD PTR C::magic_static[abi:cxx11]()::s[rip+8], 3
        mov     BYTE PTR C::magic_static[abi:cxx11]()::s[rip+19], 0
        call    __cxa_atexit
        mov     edi, OFFSET FLAT:guard variable for C::magic_static[abi:cxx11]()::s
        call    __cxa_guard_release
        jmp     .L3
guard variable for C::magic_static[abi:cxx11]()::s:
        .zero   8
C::magic_static[abi:cxx11]()::s:
        .zero   32
```

Now the code returns a cached reference to the variable s. As a result, the compiler has eliminated all of these calls to magic_static_ref.size() because it knows that there are no side effects with those calls. Running `valgrind --tool=callgrind`, I was able to quantify that number of executions for the cache reference version was about 2.5 times less than the non-cached version. 

# lesson:
If working with static values that needs to be accessed many times, consider cacheing them.