# DRY Copy Assignment

In C++ creating a declaring a copy constructor along does not delete the copy assignment operator, so it's easy to end up with a user defined copy constructor with a default-generated assignment. Unfortunately, this default assignment operator doesn't have any way to delegate to the custom copy constructor, so it still performs a byte-wise (shallow) copy no matter what. This is where the "Rule of 3"[[1](http://en.cppreference.com/w/cpp/language/rule_of_three)] comes from, which is a great start.

## Typical Rule of 3 Implementation
```C++
class TreeNode
{
public:

    TreeNode()
    :left(nullptr)
    ,right(nullptr)
    {

    }

    TreeNode(TreeNode const& other)
    :left(other.left ? new TreeNode(*left) : nullptr)
    ,right(other.right ? new TreeNode(*right) : nullptr)
    {

    }

    TreeNode& operator=(TreeNode const& other)
    {
        delete std::exchange(left, nullptr);
        delete std::exchange(right, nullptr);

        if(other.left)
            left = new TreeNode(*other.left);

        if(other.right)
            right = new TreeNode(*other.right);

        return *this;
    }

    ~TreeNode()
    {
        delete left;
        delete right;
    }

private:
    TreeNode* left;
    TreeNode* right;
}
```

Note: `std::exchange(A,B)` assigns `B` to `A`, but returns the original value, hence we can use it with our delete statement to do the assignment and destruction in one statement.

This works great on execution -- the copy constructor creates deep copies, the destructor cleans up after the class, and the assignment does both. But, if we ever change the copy constructor, or the destructor, we'll have to change the assignment operator...

## DRY Implementation

In the last section we said that "... the assignment [operator] does both.", which is exactly what we really want; we want to perform the operations of the destructor, then perform the operations of the constructor. As it turns out, we can do exactly that, but first, some theory.

In C++ every object essentially has two low-level properties -- allocation, and initialization. In order to use an object, it must first be allocated, then initialized. To stop using an object, you must uninitialize it, and then deallocate it. There are two pairs of functions that accomplish this -- the `operator new` allocates memory, and the constructor initializes the object. Then the destructor uninitializes the object, and the `operator delete` deallocates the memory.

When using an object, it must be allocated, and initialized, it is undefined behavior to use an object that is not both. Furthermore, there are consequences to allocating an object that is already allocated, or initializing an object that is already initialized, so that needs to be avoided too.

Fortunately, the following snippet covers all of these cases
```C++
TreeNode& operator=(TreeNode const& other)
{
    this->~TreeNode();
    new(this) TreeNode(other);
    return *this;
}
```

The second line is definitely a little weird looking, so let's break it down a little bit. As it turns out, the new-expression in C++ is allowed to take arguments (in this case, `this`), which it can use to forward to `operator new`. Usually it takes no parameters, and the empty parenthesis can be left out, however in this case we use the 'placement new' syntax. Placement new is a special overload of `operator new` that does no allocation, allowing the compiler to simply initialize an object in a specific place.

Putting it all together, we explicitly invoke the destructor, which leaves the object in an uninitialized, but allocated state. From here we invoke the placement new, which skips allocation and initializes our object via copy-construction. Finally, we return ourself for standard assignment chaining. In this way, we don't have any code repetition, which gives us all of the benefits of the DRY principle.

# References

1. http://en.cppreference.com/w/cpp/language/rule_of_three
