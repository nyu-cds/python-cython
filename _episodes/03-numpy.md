---
title: "Cython for NumPy Users"
teaching: 20
exercises: 0
questions:
objectives:
keypoints:
---
NumPy can be used from Cython in exactly the same manner as in regular Python, however Cython also has a number of features that support 
fast access to NumPy arrays that can result in significant performance gains. In this section, we will look at how some of these features 
can be used.

The code below does 2D discrete convolution of an image with a filter.

~~~
import numpy as np
def naive_convolve(f, g):
    # f is an image and is indexed by (v, w)
    # g is a filter kernel and is indexed by (s, t),
    #   it needs odd dimensions
    # h is the output image and is indexed by (x, y),
    #   it is not cropped
    if g.shape[0] % 2 != 1 or g.shape[1] % 2 != 1:
        raise ValueError("Only odd dimensions on filter supported")
    # smid and tmid are number of pixels between the center pixel
    # and the edge, ie for a 5x5 filter they will be 2.
    #
    # The output size is calculated by adding smid, tmid to each
    # side of the dimensions of the input image.
    vmax = f.shape[0]
    wmax = f.shape[1]
    smax = g.shape[0]
    tmax = g.shape[1]
    smid = smax // 2
    tmid = tmax // 2
    xmax = vmax + 2*smid
    ymax = wmax + 2*tmid
    # Allocate result image.
    h = np.zeros([xmax, ymax], dtype=f.dtype)
    # Do convolution
    for x in range(xmax):
        for y in range(ymax):
            # Calculate pixel value for h at (x,y). Sum one component
            # for each pixel (s, t) of the filter g.
            s_from = max(smid - x, -smid)
            s_to = min((xmax - x) - smid, smid + 1)
            t_from = max(tmid - y, -tmid)
            t_to = min((ymax - y) - tmid, tmid + 1)
            value = 0
            for s in range(s_from, s_to):
                for t in range(t_from, t_to):
                    v = x - smid + s
                    w = y - tmid + t
                    value += g[smid - s, tmid - t] * f[v, w]
            h[x, y] = value
    return h
~~~
{: .python}

Let's get a baseline on how fast this code executes:

~~~
%timeit naive_convolve(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
The slowest run took 79.78 times longer than the fastest. This could mean that an intermediate result is being cached.
10000 loops, best of 3: 28.7 µs per loop
~~~
{: .python}

As we saw previously, we can simply compile this code using Cython and expect some performance improvements. Let's copy the 
`naive_convolve` function from the cell above, then rename the function `convolve1` and compile it with Cython. 

~~~
%%cython
import numpy as np
def convolve1(f, g):
    # f is an image and is indexed by (v, w)
    # g is a filter kernel and is indexed by (s, t),
    #   it needs odd dimensions
    # h is the output image and is indexed by (x, y),
    #   it is not cropped
    if g.shape[0] % 2 != 1 or g.shape[1] % 2 != 1:
        raise ValueError("Only odd dimensions on filter supported")
    # smid and tmid are number of pixels between the center pixel
    # and the edge, ie for a 5x5 filter they will be 2.
    #
    # The output size is calculated by adding smid, tmid to each
    # side of the dimensions of the input image.
    vmax = f.shape[0]
    wmax = f.shape[1]
    smax = g.shape[0]
    tmax = g.shape[1]
    smid = smax // 2
    tmid = tmax // 2
    xmax = vmax + 2*smid
    ymax = wmax + 2*tmid
    # Allocate result image.
    h = np.zeros([xmax, ymax], dtype=f.dtype)
    # Do convolution
    for x in range(xmax):
        for y in range(ymax):
            # Calculate pixel value for h at (x,y). Sum one component
            # for each pixel (s, t) of the filter g.
            s_from = max(smid - x, -smid)
            s_to = min((xmax - x) - smid, smid + 1)
            t_from = max(tmid - y, -tmid)
            t_to = min((ymax - y) - tmid, tmid + 1)
            value = 0
            for s in range(s_from, s_to):
                for t in range(t_from, t_to):
                    v = x - smid + s
                    w = y - tmid + t
                    value += g[smid - s, tmid - t] * f[v, w]
            h[x, y] = value
    return h
~~~
{: .python}

We should run some tests to make sure that the functions produce the same results:

~~~
from nose.tools import assert_equal
res1 = naive_convolve(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
res2 = convolve1(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
assert_equal((res1==res2).all(), True)
~~~
{: .python}

This doesn't produce any output, so that means the assertion succeeded. Now we can time the new function and see if it is any faster.

~~~
%timeit convolve1(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
10000 loops, best of 3: 19.5 µs per loop
~~~
{: .python}

So we got a speedup of 28.7 / 19.5 ~ 1.5 times. Not bad.

The next step is to add Cython data types to the code. This code will no longer be compatible with Python, so the consequences of doing this 
must be carefully considered. The most important change is to use variables that have the same data type as the elements of the NumPy arrays.

For normal variables, this simply means adding a line like `cdef type variable` (where `type` is the appropriate C type) for each variable
in the program. Since it is also possible to initialize the variable at the same time it is declared, we can simplify the code by declaring
it the first time it is used. In the `convolve1` case above, we know that the indices of the NumPy arrays are integers, so we 
can do something like:

~~~
cdef int vmax = f.shape[0]
cdef int wmax = f.shape[1]
cdef int smax = g.shape[0]
cdef int tmax = g.shape[1]
...
~~~
{: .python}

This is simpler than doing the following for each variable:

~~~
cdef int vmax
vmax = f.shape[0]
...
~~~
{: .python}

It is important to type *all* variables. Cython does not give any warnings if you do not, however the code will run much slower.

The NumPy arrays themselves also need to have type information added, and they correspond to the `np.ndarray` type. The `f` and `g` 
arrays appear as parameters to the function, so the type can be added as part of the function declaration.  The `h` array is initialized 
to zeros in the code so requires a `cdef np.ndarray` to be added as follows:

~~~
...
def convolve1(np.ndarray f, np.ndarray g):
...
cdef np.ndarray h = np.zeros([xmax, ymax], dtype=f.dtype)
...
~~~
{: .python}

The one remaining variable to be changed is `value`. We would like to use the same type as that stored in the NumPy arrays, however we
can't just use `np.int` as this is a run-time type. However, for every run-time type, there is a corresponding compile-time type that ends
in `_t`. This allows us to declare `value` as follows:

~~~
cdef np.int_t value
~~~
{: .python}

> ## Challenge
> The code below will not compile as parts have been commented out. Copy this code into a Jupyter cell, or into IPython and rename the
> function `convolve2`. Read each of the 
> commented sections for a description of how the data types are added, then uncomment the line to enable to statement. When you have 
> completed this for the whole function, run the code to ensure that it compiles correctly.
> 
> ~~~
> %%cython
> import numpy as np
> # "cimport" is used to import special compile-time information
> # about the numpy module (this is stored in a file numpy.pxd which is
> # currently part of the Cython distribution).
> #cimport numpy as np
> ​
> # "def" can type its arguments but not have a return type. The type of the
> # arguments for a "def" function is checked at run-time when entering the
> # function.
> #
> # The arrays f, g and h is typed as "np.ndarray" instances. The only effect
> # this has is to a) insert checks that the function arguments really are
> # NumPy arrays, and b) make some attribute access like f.shape[0] much
> # more efficient. (In this example this doesn't matter though.)
> def convolve2(np.ndarray f, np.ndarray g):
>     if g.shape[0] % 2 != 1 or g.shape[1] % 2 != 1:
>         raise ValueError("Only odd dimensions on filter supported")
>     # The "cdef" keyword is also used within functions to type variables. It
>     # can only be used at the top indendation level (there are non-trivial
>     # problems with allowing them in other places, though we'd love to see
>     # good and thought out proposals for it).
>     #
>     # For the indices, the "int" type is used. This corresponds to a C int,
>     # other C types (like "unsigned int") could have been used instead.
>     #cdef int vmax = f.shape[0]
>     #cdef int wmax = f.shape[1]
>     #cdef int smax = g.shape[0]
>     #cdef int tmax = g.shape[1]
>     #cdef int smid = smax // 2
>     #cdef int tmid = tmax // 2
>     #cdef int xmax = vmax + 2*smid
>     #cdef int ymax = wmax + 2*tmid
>     #cdef np.ndarray h = np.zeros([xmax, ymax], dtype=np.int)
>     #cdef int x, y, s, t, v, w
>     
>     # It is very important to type ALL your variables. You do not get any
>     # warnings if not, only much slower code (they are implicitly typed as
>     # Python objects).
>     #cdef int s_from, s_to, t_from, t_to
>     
>     # For the value variable, we want to use the same data type as is
>     # stored in the array, so we use "np.int_t".
>     # NB! An important side-effect of this is that if "value" overflows its
>     # datatype size, it will simply wrap around like in C, rather than raise
>     # an error like in Python.
>     #cdef np.int_t value
>     
>     for x in range(xmax):
>         for y in range(ymax):
>             s_from = max(smid - x, -smid)
>             s_to = min((xmax - x) - smid, smid + 1)
>             t_from = max(tmid - y, -tmid)
>             t_to = min((ymax - y) - tmid, tmid + 1)
>             value = 0
>             for s in range(s_from, s_to):
>                 for t in range(t_from, t_to):
>                     v = x - smid + s
>                     w = y - tmid + t
>                     value += g[smid - s, tmid - t] * f[v, w]
>             h[x, y] = value
>     return h
> ~~~
> {: .python}
>
> Next, run the following test to make sure the result is the same:
>
> ~~~
> from nose.tools import assert_equal
> res1 = naive_convolve(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
> res2 = convolve2(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
> assert_equal((res1==res2).all(), True)
> ~~~
> {: .python}
>
> Now time this code and see if it has improved.
>
> ~~~
> %timeit convolve2(np.array([[1, 1, 1]], dtype=np.int), np.array([[1],[2],[1]], dtype=np.int))
> ~~~
> {: .python}
>
> Did the time improve?
{: .challenge}

### Efficient Indexing

This code is still not as efficient as it could be. Array lookups and assignments, like those using the `[]`-operator, still uses full Python 
operations. It would be much more effient if we could access the data buffer directly at C speed.

It is possible to do this by specifying the type of *contents* of the `ndarray` objects. We do this with a special “buffer” syntax which must 
be told the datatype (first argument) and number of dimensions (“ndim” keyword-only argument, if not provided then one-dimensional is assumed).

The format for this syntax is:

~~~
np.ndarray[type, ndim=N]
~~~
{: .python}

Here, `type` is the compile-time type of the array elements, and `N` is the number of dimensions in the array. This syntax is used wherever 
an `ndarray` is declared.

In the `convolve2` example, we would need to use this in two places as follows:

~~~
...
def convolve2(np.ndarray[np.int_t, ndim=2] f, np.ndarray[np.int_t, ndim=2] g):
...
cdef np.ndarray[np.int_t, ndim=2] h = ...
~~~
{: .python}

### More Indexing Improvements

The NumPy array lookups are still slowed down by two other factors:

* Bounds checking is performed.
* Negative indices are checked for and handled correctly.

If we are certain that code will always access within the array bounds, and that it doesn’t use negative indices, then it is possible to 
obtain some extra performance by avoiding these checks.

*Note however that this comes at the cost of safety.* If the code does not behave exactly as you expect, it could crash the program or 
corrupt data.

Bounds checking can be disabled by adding a decorator to the function as follows:

~~~
...
cimport cython
@cython.boundscheck(False) # turn off bounds-checking for entire function
def convolve2(np.ndarray[np.int_t, ndim=2] f, np.ndarray[np.int_t, ndim=2] g):
...
~~~
{: .python}

Now bounds checking is not performed. It is possible to switch bounds-checking mode in many ways, see 
[Compiler directives](http://docs.cython.org/src/reference/compilation.html#compiler-directives) for more information.

Negative indices are dealt with by forcing the indexes to be positive using the unsigned integer type for the index variables and 
casting values to this type. If negative values are cast, then this will create a very large positive value instead and it may result 
in an attempt to access out-of-bounds values. Casting is done with a special `<>`-syntax. The code below shows how to change the 
function to use either unsigned ints or casting as appropriate:

~~~
...
cdef int s, t
cdef unsigned int x, y, v, w
...
               v = <unsigned int>(x - smid + s)
               w = <unsigned int>(y - tmid + t)
               value += g[<unsigned int>(smid - s), <unsigned int>(tmid - t)] * f[v, w]
...
~~~
{: .python}

> ## Challenge
> 
> Make the changes suggested in the previous sections to `convolve2` and see how much the performance improves.
{: .challenge}