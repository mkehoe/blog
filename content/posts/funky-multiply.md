---
date: "2020-09-17"
title: "Funky Multiply in C"
slug: "funky-multiply"
tags: [
    "c",
    "asm"
]
categories: [
    "c"
]
disable_comments: true
---

A few weeks ago I saw an ingenious way to multiply in C and can't stop thinking about it. If you want to write insane code to trip up a code reviewer up, here is some bizarre code for a multiplication:

```
int p = sizeof(char[a][b]);
```

There is a couple mildly interesting things about this. First, this will not compile in Visual Studio. Array declarations with non-consts is part of the C99 spec, but VS only supports either C89/C90 or C99 features also defined in the C++ standard.

This does compile fine with the clang and gcc compilers. With the x64-86 gcc 10.2 the asm generated looks like this:

```
mov     edx, DWORD PTR [rbp-24]
mov     eax, DWORD PTR [rbp-20]
imul    eax, edx
mov     DWORD PTR [rbp-4], eax
```

And that looks eerily similar to the standard way to multiply:

```
mov     eax, DWORD PTR [rbp-20]
imul    eax, DWORD PTR [rbp-24]
mov     DWORD PTR [rbp-4], eax
```

Interestingly enough, the compiler is smart enough to use the same imul instruction for both of these.


PS. Shout out to godbolt.org and the great work they have done in making it easy to see how code compiles to asm.
