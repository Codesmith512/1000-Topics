# Overview  
This page contains a list of some generic tricks in c++ to aide with readability. This is not a coding standards doc, so it won't mention anything to do with comments, code documentation, etc. Instead, it focuses on core c++ features that can be used in addition to coding style to make thing more readable.

## Aggregate Initialization [[1](http://en.cppreference.com/w/cpp/language/aggregate_initialization)]

Aggregate initialization is a special type of list initialization (see below) that can be used to quickly, and efficiently initialize certain c++ classes. To use it, simply assign/construct your object with braces instead, and provide the values for all member variables in order of declaration inside them. It has many restrictions on it's use, and is most compatible (and useful) with generic structs, as it provides a convenient, one-line way to initialize them.  

For example, `struct Login { std::string username, password };` can be easily initialized with `Login l = {"codesmith512", "password123"};` (Please do not store passwords in plaintext in your program). I use this technique a lot because these types of objects are much more readable than `std::pair<...>` or `std::tuple<...>` for a function that returns multiple things (or helper classes in general), and aggregate initialization makes initializing them easy without specifying a lengthy constructor, or initializing each member variable on a new line.

## List Initialization [[2](http://en.cppreference.com/w/cpp/language/list_initialization)]

List initialization is very common to explicitly initialize arrays as such: `int array[] = { 1, 2, 3, 4, 5 };`. It is also fairly common to see other objects initialized in this manner too: `std::vector<int> array = { 1, 2, 3, 4, 5 };`. However, a very powerful use I've found for this tool is in return expressions. Complex return expressions can be quickly reduced because the type isn't required to be specified. For example:
```
std::vector<std::vector<std::vector<int>>> get3dVector()
{
  return std::vector<std::vector<std::vector<int>>>();
}
```
can quickly become
```
std::vector<std::vector<std::vector<int>>> get3dVector()
{
  return {};
}
```
Now that's convenient!

Even better, returning a 1x1x1 vector of 0 would typically be one of two (or probably more) things:
```
return std::vector<std::vector<std::vector<int>>>(1, std::vector<std::vector<int>>(1, std::vector<int>(1, 0)));
```
or
```
std::vector<std::vector<std::vector<int>>> myReturn;
myReturn.emplace_back();
myReturn[0].emplace_back();
myReturn[0][0].emplace_back(0);
return myReturn
```

However, with list initialization, the impossible is easy!
```
return {{{0}}};
```

# References
1. http://en.cppreference.com/w/cpp/language/aggregate_initialization
2. http://en.cppreference.com/w/cpp/language/list_initialization