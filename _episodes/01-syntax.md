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

Integer
: C provides a variety of integer types that vary primarily in the size of the integer that can be represented, and if the integer can represent
negative numbers, or only positive numbers. The size is determined by the number of bits used to store the integer, ranging from 8 bits (512 distinct
values) to 64 bits (2<sup>64</sup> - 1 distinct values). One bit is usually dedicate to representing negative numbers, so the maxium signed integers 
are half their unsigned counterparts. The following table shows the statements for introducing different integers:

<table border="1">
<tr><th>Type</th><th>Description</th><tr>
<tr><td><code>char</code></td><td>8-bit signed integer</td></tr>
<tr><td><code>short</code></td><td>16-bit signed integer</td></tr>
<tr><td><code>int</code></td><td>32-bit signed integer</td></tr>
<tr><td><code>long</code></td><td>64-bit signed integer</td></tr>
<tr><td><code>long long</code></td><td>64-bit signed integer</td></tr>
</table>

Floating Point
: Floating point is a means of representing decimal numbers. In C, these are primarily distinguished by the largest (and smallest) numbers that
can be represented, which is again a factor of the number of bits used. The following table shows the statements for introducing different 
floating point types:

<table border="1">
<tr><th>Type</th><th>Description</th><tr>
<tr><td><code>float</code></td><td>32-bit floating point</td></tr>
<tr><td><code>double</code></td><td>64-bit floating point</td></tr>
<tr><td><code>long double</code></td><td>80-bit floating point</td></tr>
</table>

C also provides a number of structured data types that can be used to build on these fundamental types. These include:

Array
: The array represents a sequence of values that can be accessed using a base address and index. C supports multi-dimensional arrays. 
The statement for introducing an array of `size` elements, each of type `type`, is:

~~~
type name[size]
~~~
{: .code}

Pointer
: A pointer is a high level representation of an address in the computer's memory. The statement for introducing a pointer to an object of
type `type` is:

~~~
type *name
~~~
{: .code}

Structure
: Structures provide a mechanism for grouping data together into a more convenient form. The layout of the the data in a structure, and 
correspondingly how it will be represented in memory, can be very closely controlled. The statement for introducing a structure is:

~~~
struct name { declaration }
~~~
{: .code}

Union
: A union is means of providing multple representations of the same data. The statement for introducing a union is:

~~~
union name { declaration }
~~~
{: .code}

Enumeration
: An enumeration provides a user-defined type that consists of integral constants. The constants are represented using symbolic names in the program.
The statement for introducing an enumeration is:

~~~
enum name { declaration }
~~~
{: .code}