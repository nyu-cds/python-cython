---
layout: lesson
root: .
---
Cython is a modification of Python that adds C data types. Almost any piece of Python code is also valid Cython code (with a few limitations.) 
Cython then converts the (modified) Python code into C code which makes equivalent calls to the Python/C API. This C code is then compiled 
into a shared library which can be imported into Python.

In Cython, function parameters and variables can be declared to have C data types, and code which manipulates Python values and C values can 
be freely intermixed. Cython takes care of converting from C to Python data types automatically wherever possible. Reference count maintenance 
and error checking of Python operations is also automatic, and the full power of Python’s exception handling facilities, including the 
try-except and try-finally statements, is still available.

There are two main benefits of Cython:

Speed
: How much performance improves depends very much on the program. Typical Python numerical programs would tend to gain very little as most 
time is spent in lower-level C anyway. However, for-loop-style programs can improve by many orders of magnitude.

Easy calling into C code
: One of Cython’s purposes is to allow easy wrapping of C libraries. When writing code in Cython you can call into C code as easily as into 
Python code.

The following sections provide a very brief introduction to Cython. See Cython Language Basics for a more detailed description of the Cython 
language.

> ## Prerequisites
>
> The examples in this lesson can be run directly using the Python interpreter, using IPython interactively, 
> or using Jupyter notebooks.
{: .prereq}

