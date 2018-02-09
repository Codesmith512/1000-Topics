# CMake Windows Library Formats

In CMake, under windows (I don't have any experience with other platforms and CMake), there are two main kinds of libraries, shared libraries and static libraries. Put simply, static libraries gets compiled and linked into your program at the same time, where shared libraries get linked at compiled time, but are brought into memory for use separately at runtime.

## Compiler Theory

For the purpose of this article, some minimal compiler theory is required, so make sure you're familiar with the concepts in `/C++/Basic Compiler-Linker Theory.md`

## Static Libraries (.lib)

Static libraries are basically the result of compiling the source files, and just tossing all of the objects into one big file called a library file. Then to use them, your project compiles all of it's object files, and links them normally with the library file included. This has the upside of being very nearly the exact same process as normal, making it simple and easy. The downside is that if two programs/shared libraries/etc. need to reference the same static library, it gets included one time in each, taking up more space.

## Shared Libraries (.dll + .lib)

Shared libraries attempt to solve the drawback of the size of static libraries, by instead doing a partial link on link-time, and finishing the linking on program load. This allows multiple programs/modules to _share_ the library, because the all link to the same instance on runtime.

Compiling a shared library results in two files, the .lib from before, and a .dll. The purpose of this .lib however, is to provide which symbols are defined in the .dll and where, so the compile-time link phase can complete. The actual definitions are stored in the .dll, and the runtime link phase finishes the work by loading the .dll into memory, and replacing the lookup values from the .lib.

## Shared Libraries in MSVC++

The problem with shared libraries in C++ is that sometimes the definitions will be provided (if you're making the DLL), and sometimes they shouldn't be (if you're using the DLL). To tackle this, Windows adds two `__declspec` attributes[[1](https://msdn.microsoft.com/en-us/library/3y1sfaz2.aspx)].

`__declspec(dllimport)`
    - Any symbols with this attribute _cannot_ have a definition
        + as such, they compile perfectly fine without one
    - Symbols will be read from the .dll/.lib, and must be present there 
`__declspec(dllexport)`
    - Any symbols with this attribute _must_ have a definition
        + they will compile fine without one, however anybody using such a symbol will get errors
    - Symbols will be written to the .dll/.lib

Using these attributes:

`__declspec(dllimport) static double start_time;`
- Declares a `static double start_time` that resides in an external dll 
- Note the lack of a definition

`__declspec(dllexport) static void foo() { std::cout << "Hello World"; }`
- Declares a `static void foo()` that resides in this dll
- Definition is provided, and the symbol is added to the DLL lookup

```
class __declspec(dllimport) MyTimer
{
    struct __declspec(dllimport) state
    {
        void update();
        bool is_valid();
    }

    void start();
    double elapsed();
    double reset();
}
```
- The `__declspec(dllimport)` tag in a class applies to all member functions and variables, but not to nested types (these just need another specifier).

## CMake

To link against any library, simply add it to the project's `target_link_libraries`. You can even mix static and shared libraries!

To assist with the help of making shared libraries, CMake adds a definition for a value named `${PROJECT}_EXPORTS` to the current project it's compiling (if it is a shared library). Typically, this is leveraged to make header files that work for both importing and exporting DLL symbols[[2](https://cmake.org/cmake/help/v3.0/prop_tgt/DEFINE_SYMBOL.html)], see below:

```
#ifdef Project_EXPORTS
#define PROJ_API __declspec(dllexport)
#else
#define PROJ_API __declspec(dllimport)
#endif
```

This way, you simply use `PROJ_API` instead of dllimport/dllexprt, and when it's compiling a source file in the project, you get the export tag; otherwise you get the import tag.

# References
1. https://msdn.microsoft.com/en-us/library/3y1sfaz2.aspx
2. https://cmake.org/cmake/help/v3.0/prop_tgt/DEFINE_SYMBOL.html
