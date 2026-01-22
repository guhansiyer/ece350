# Dynamic Memory Allocation

In Java, the `new` keyword is used to create a new instance of an object. The runtime will garbage collect the object later on.

In C++, `new` and `delete` are used to allocate and deallocate objects respectively.

In C, memory allocation is done on a lower level with `malloc()` and `free()`.

For example, to allocate an integer, you call `malloc(sizeof(int))`. This creates a new integer in memory and returns its address, which can be stored in a pointer.

When `free()` is called on a pointer, the memory it occupied is marked as available, but might not be cleared or reused immediately after.

## Fulfilling Memory Requests

Note that `free()` does not specify how much memory is returned. This means that (a) the OS keeps track of each allocated block's size and (b) it is not possible to return part of a block.

The operating system will try to find some free memory to meet a request.

Although running out of memory in modern computers is a rare occurence, there is the possibility that some request may not be fulfilled because no block meeting the need of the request is available.