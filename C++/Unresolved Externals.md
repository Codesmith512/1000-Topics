# Unresolved Externals (UEs)
Unresolved externals in C++ seem to be one of the trickiest errors that's commonly encountered. I personally attribute this to the fact that this particular error is emitted by the linker, not the compiler, so the code is "perfectly valid" but fails anyway. 

## Compiler Theory

For the purpose of this article, some minimal compiler theory is required, so make sure you're familiar with the concepts in `/C++/Basic Compiler-Linker Theory.md`

## Underlying Cause

Ultimately, it doesn't matter what you did in code, there is exactly one thing that will result in an unresolved external: there is no definition for a particular symbol the linker is trying to reference.

## Apparent Causes

The first step is to identify what you're looking for -- every compiler I've used will inform you of the symbol that could not be found, in addition to the function using that symbol. They're not perfectly human-readable, due to name mangling, however it is usually close enough to figure out what you're looking for.

### Functions
- Ensure the function is properly exported, or in a class that is exported
- Make sure the function has a definition
    + Usually this is a compile error in the library that made the function. However compile errors are not generated on things that aren't used, so if the library never used it, it won't generate a compile error, but it won't have a symbol definition to use.

### Static Variables
- Make sure the variable has a definition (ie: `static int i;` in the header needs an `int i = 0;` in the source in a global location)
    + Unlike the function version, this will always throw an unresolved external.

### Classes
Classes never throw unresolved externals themselves, but if you notice that everything in a class has them, check these...
- Make sure the class is properly exported (in windows `__declspec(dllimport/dllexport)`)
    + _Keep in mind that nested classes need their own export tags_

### Libraries/Projects
Again, libraries never throw unresolved externals themselves, but if you notice that everything in a library has them, check these...
- Make sure the library is actually being linked against...
    + In CMake, make sure it is listed in `target_link_libraries(...)`
    + In MSVC, make sure it is listed in `Project -> Properties -> Linker -> Input -> Additional Inputs`

If this didn't help, msdn has an article that includes some less common causes. [1](https://msdn.microsoft.com/en-us/library/799kze2z.aspx)

# References
1. https://msdn.microsoft.com/en-us/library/799kze2z.aspx
