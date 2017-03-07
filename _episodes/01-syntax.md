---
title: "Cython Syntax"
teaching: 20
exercises: 0
questions:
objectives:
keypoints:
---
### Basic C Types

The C language provides a number of basic data types, and many of these are available in Cython. As C is designed for system-level programming,
many of the data types mape closely to the format of the data that is represented by the hardware. There are two fundamental C data types:

#### Integer

C provides a variety of integer types that vary primarily in the size of the integer that can be represented, and if the integer can represent
negative numbers, or only positive numbers. The size is determined by the number of bits used to store the integer, ranging from 8 bits (512 distinct
values) to 64 bits (2<sup>64</sup> - 1 distinct values). One bit is usually dedicate to representing negative numbers, so the maxium signed integers 
are half their unsigned counterparts. The following table shows the statements for introducing different integers:

<table border="1">
<tr><th>Type</th><th>Description</th></tr>
<tr><td><code>char</code></td><td>8-bit signed integer</td></tr>
<tr><td><code>short</code></td><td>16-bit signed integer</td></tr>
<tr><td><code>int</code></td><td>32-bit signed integer</td></tr>
<tr><td><code>long</code></td><td>64-bit signed integer</td></tr>
<tr><td><code>long long</code></td><td>64-bit signed integer</td></tr>
</table>

#### Floating Point

Floating point is a means of representing decimal numbers. In C, these are primarily distinguished by the largest (and smallest) numbers that
can be represented, which is again a factor of the number of bits used. The following table shows the statements for introducing different 
floating point types:

<table border="1">
<tr><th>Type</th><th>Description</th></tr>
<tr><td><code>float</code></td><td>32-bit floating point</td></tr>
<tr><td><code>double</code></td><td>64-bit floating point</td></tr>
<tr><td><code>long double</code></td><td>80-bit floating point</td></tr>
</table>

C also provides a number of structured data types that can be used to build on these fundamental types. These include:

#### Array

The array represents a sequence of values that can be accessed using a base address and index. C supports multi-dimensional arrays. 
The statement for introducing an array of `size` elements, each of type `type`, is:

~~~
type name[size]
~~~
{: .code}

#### Pointer

A pointer is a high level representation of an address in the computer's memory. The statement for introducing a pointer to an object of
type `type` is:

~~~
type *name
~~~
{: .code}

#### Structure

Structures provide a mechanism for grouping data together into a more convenient form. The layout of the the data in a structure, and 
correspondingly how it will be represented in memory, can be very closely controlled. The statement for introducing a structure is:

~~~
struct name { declaration }
~~~
{: .code}

#### Union

A union is means of providing multple representations of the same data. The statement for introducing a union is:

~~~
union name { declaration }
~~~
{: .code}

#### Enumeration

An enumeration provides a user-defined type that consists of integral constants. The constants are represented using symbolic names in the program.
The statement for introducing an enumeration is:

~~~
enum name { declaration }
~~~
{: .code}

### Variable and Type Definitions

The `cdef` statement is used to declare C variables, either local or module-level:

~~~
cdef int i, j, k
cdef float f, g[42], *h
~~~
{: .python}

In C, types can be given names using the `typedef` statement. The equivalent in Cython is `ctypedef`:

~~~
ctypedef int * intPtr
~~~
{: .python}

Cython also supports C struct, union, or enum types:

<table border="1">
<tr><th>C code</th><th>Cython code</th></tr>
<tr><td><pre><code>
struct Grail {
    int age;
    float volume;
}	cython
</code></pre></td>
<td><pre><code>
cdef struct Grail:
    int age
    float volume
</code></pre></td></tr>
<tr><td><pre><code>
union Food {
    char *spam;
    float *eggs;
}	
</code></pre></td>
<td><pre><code>
cdef union Food:
    char *spam
    float *eggs
</code></pre></td></tr>
<tr><td><pre><code>
enum CheeseType {
    cheddar, edam,
    camembert
}	
</code></pre></td>
<td><pre><code>
cdef enum CheeseType:
    cheddar, edam,
    camembert
</code></pre></td></tr>
<tr><td><pre><code>
emum CheeseState {
    hard = 1,
    soft = 2,
    runny = 3
}	
</code></pre></td>
<td><pre><code>
cdef enum CheeseState:
    hard = 1
    soft = 2
    runny = 3
</code></pre></td></tr>
</table>

### Functions

There are two kinds of function definition in Cython:

Python functions
: These are defined using the `def` statement, as in Python. They take Python objects as parameters and return Python objects.
C functions 
: These are defined using the new `cdef` statement. They take either Python objects or C values as parameters, and can return either Python objects or C values.

Within a Cython module, Python functions and C functions can call each other freely, but only Python functions can be called from outside the 
module by interpreted Python code. So, any functions that you want to “export” from your Cython module must be declared as Python functions 
using `def`. There is also a hybrid function, called `cpdef`, that can be called from anywhere, but uses the faster C calling conventions when 
being called from other Cython code. A `cpdef` can also be overridden by a Python method on a subclass or an instance attribute, even when 
called from Cython. If this happens, most performance gains are of course lost and even if it does not, there is a tiny overhead in calling 
a `cpdef` method from Cython compared to calling a `cdef` method.

Parameters of either type of function can be declared to have C data types, using normal C declaration syntax. For example:

~~~
def spam(int i, char *s):
    ...

cdef int eggs(unsigned long l, float f):
    ...
~~~
{: .python}

Automatic conversion is currently only possible for numeric types, string types and `structs` (composed recursively of any of these types); 
attempting to use any other type for the parameter of a Python function will result in a compile-time error. Care must be taken with strings 
to ensure a reference if the pointer is to be used after the call. Structs can be obtained from Python mappings, and again care must be 
taken with string attributes if they are to be used after the function returns.

C functions, on the other hand, can have parameters of any type, since they’re passed in directly using a normal C function call.

Functions declared using `cdef`, like Python functions, will return a `False` value when execution leaves the function body without an explicit 
return value. This is in contrast to C/C++, which leaves the return value undefined.

### Automatic Type Conversions

In most situations, automatic conversions will be performed for the basic numeric and string types when a Python object is used in a context 
requiring a C value, or vice versa. The following table summarises the conversion possibilities.

<table border="1">
<tr><th>C types</th><th>From Python types</th><th>To Python types</th></tr>
<tr><td>char, short, int, long</td><td>int, long</td><td>int</td></tr>
<tr><td>int, long, long long</td><td>int, long</td><td>long</td></tr>
<tr><td>float, double, long double</td><td>int, long, float</td><td>float</td></tr>
<tr><td>char*</td><td>str</td><td>str</td></tr>
<tr><td>struct, union</td><td></td><td>dict</td></tr>
</table>

### Statements and Expressions

Control structures and expressions follow Python syntax for the most part. When applied to Python objects, they have the same semantics as 
in Python (unless otherwise noted). Most of the Python operators can also be applied to C values, with the obvious semantics.

If Python objects and C values are mixed in an expression, conversions are performed automatically between Python objects and C numeric 
or string types.
