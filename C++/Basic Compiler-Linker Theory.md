# Basic C++ Compiler-Linker Theory

Some articles require a basic knowledge of the C++ compiler and linker. If you've been linked here, chances are you were reading an article that does.

Feel free to skip this article if
- You've already read it and remember it
- You actually care about compiler theory
    + This article contains over-simplifications, so please visit the references at the bottom of this article instead, or do some other research to gain this information correctly.
- You already know compiler theory
    + By all means, skip this article, you don't need it!
- You think you already know compiler theory
    + Feel free to skip this article and come back to it for reference if you need to!

## Overview

For the purpose of this article, the compiler effectively has two parts -- the compiler, and the linker (and they run in that order). The compiler translates almost all source code to binary/assembly/etc. , but leaves symbol references in place of function calls and variable references, and provides a symbol definition where they are defined. It outputs the result to an object file.

## Compiler

The source file `first.cpp`
```
static int foo = 4;
static int bar()
{ return 0; }
```
can conceptually be thought of as being compiled into `first.obj`
```
global foo:
    4
global bar__void:
    return 0
```

Another source file `second.cpp`
```
#include "first.h"
static int print_4()
{ std::cout << (bar() + foo); }
```
can be thought of as compiling into `second.obj`
```
// first.h only has declarations for foo/bar, no code is produced

global print_4__void:
    @cout__operator<<__int(@bar__void() + @foo)
    return
```

## Linker

The linker then links these files together, assigning each symbol (foo, bar__void, print_4__void, etc) an address, and inserting it's definition. Then when it encounters a reference to that symbol (@foo, etc.) it can replace it with it's assigned address.

```
<data section>
0x1000: 0   // foo
...
<code section>
0x2000: return 0    // bar
0x2008: ...         // cout.<<() definition
0x2100: 0x2008(0x2000() + 0x1000)   // print_4 definition
        return
...
```

# References
[... whoops, nothing here yet ... ](https://github.com/Codesmith512/1024-Topics/issues/3)
