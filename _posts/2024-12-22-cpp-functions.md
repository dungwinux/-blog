---
layout: post
title: "Unexpected ways you can write a function in C++"
date: 2024-12-22 23:20:00 +0000
categories: code
tags: software, c++
excerpt: It's like you just discovered a new language named C++
---

If you are an experienced C++ programmer, this is undeniably the way to write a function:

```cpp
int add(int a, int b) {
    return a + b;
}
```

C++ is a statically typed language, and you can in fact type a function. Like how `qsort` function wants `comp` function for compare, you can write:

```cpp
typedef int (*binary_fun)(int, int);
int do_math(binary_fun fun, int a, int b) {
    return fun(a, b);
}
int main() {
    do_math(add, 1, 2);     // return 3
}
```

But note that in C and C++, you can cast to a type. Which means I can cast an ASCII string into a function:


```cpp
// Non-portable. x86-64 only
#define EXECPERM __attribute__((section(".text")))
#ifdef __linux__ 
char const add[] EXECPERM = "\x8d\x04\x37\xc3";
#elif _WIN32
char const add[] EXECPERM = "\x8d\x04\x0a\xc3";
#endif
```

or even from an array into a function:

```cpp
// Non-portable. x86-64 Little-endian only
#ifdef __linux__ 
long const add[] EXECPERM = {0xc337048d};
#elif _WIN32
long const add[] EXECPERM = {0xc30a048d};
#endif
```

And it should function like an `add` function given the variable is in an executable region (else you will be stopped by NX bit or similar). This is extremely useful if you want to embed one whole function in assembly and do not want to deal with GNU as, MASM, etc. I used this in one of my challenges on MinutemanCTF Training Platform as you could see here: [dungwinux:minuteman24-rev:inception/inception.c](https://github.com/dungwinux/minuteman24-rev/blob/master/inception/inception.c)

On the other hand, say, you want instead a function that holds and preserves the variables. Tranditionally, this can be done through static variables.

```cpp
int get_id() {
    static int counter = 0;
    return counter++;
}
int main() {
    get_id();
    get_id();
    return get_id();    // return 2
}
```

Thanks to operator overloading, we can construct an object that behave like a function and can preserve variables:

```cpp
struct GetId {
    int counter = 0;
    int operator()() {
        return counter++;
    }
};
auto get_id = GetId{};
int main() {
    get_id();
    get_id();
    return get_id();    // return 2
}
```

You may notice that the closure of the "function" is much narrower. The static variable `counter` is preserved across program (any call inside program will change the same variable), while the struct property `counter` is preserved across the object lifetime (calls to two different instances of `GetId` will change different `counter`). This is also different from lambda, since it owns `counter` rather than inheriting `counter` of the parent (lambda-capturing). You can find this pattern widely used in the `std::ranges` namespace implementation (for example, `std::ranges::join_view` in [MSVC STL](https://github.com/microsoft/STL/blob/90820002693fe6eaaec2e55884472c654186207e/stl/inc/ranges#L2633), [LLVM libc++](https://github.com/llvm/llvm-project/blob/d32509928ba6b4c78b02b8a8499dce056ae6fe52/libcxx/include/__ranges/join_view.h#L72), [GCC libstdc++](https://github.com/gcc-mirror/gcc/blob/2a474c28e573b8604b5fa2584f276d7b7b584cde/libstdc%2B%2B-v3/include/std/ranges#L2885)).
So don't limit yourself to preserving variables, since you can do a lot more when it is in fact a `struct`.
