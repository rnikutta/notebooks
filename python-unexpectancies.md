
# Python unexpectancies

A notebook with examples of Python pitfalls (both rookie mistakes and properly
unpleasant surprises)

## The identity operator `is`

Consider this:


    a = 256; b = 256
    a is b




    True



So far so good. But then:


    a = 257; b = 257
    a is b




    False



**Careful!** Python's identity operator `is` compares object identity, not their
value. Only if two names refer to the same item in memory, `is` evaluates to
`True`. As it happens, Python keeps a funny / stupid list of integers between -5
and +256 in memory, and for these integers always returns a reference. For other
values this is not guaranteed... Read this bit: [https://docs.python.org/2/c-api
/int.html](https://docs.python.org/2/c-api/int.html)

**Lesson learnt:** always use the `==` operator if you want to compare by value.
`is` is for instance good for comparing things to [built-in
constants](https://docs.python.org/2/library/constants.html)


    c, d = None, True
    (c is None, d is None, c is True, d is True)




    (True, False, False, True)



## References are not copies

Consider this


    list1 = ['a','b','c'] # set up a list
    list2 = list1         # set up another; are list1 and list2 separate objects?
    print list1
    print list2

    ['a', 'b', 'c']
    ['a', 'b', 'c']


They look the same. Let's change one element in `list1`


    print list1[1]

    b



    list1[1] = 'x'
    print list1

    ['a', 'x', 'c']


As expected. Observe what happened to `list2` at the same time:


    print list2

    ['a', 'x', 'c']


By saying `list1 = list2` above, both lists refer to the same piece of memory.
They are not distinct objects!
To make them distinct, and explicit copy of the values is necessary when
creating `list2`:


    list2 = list1[:]
    print list1
    print list2

    ['a', 'x', 'c']
    ['a', 'x', 'c']


Now they still look identical, but if we change one element in `list2`...


    list1[1] = 'y'
    print list1
    print list2

    ['a', 'y', 'c']
    ['a', 'x', 'c']


... then `list2` is not affected, b/c both lists now point to entirely different
places in memory.

The same does not apply for single-value variables:


    a = 3
    b = a
    print a,b

    3 3



    b = 7
    print a,b

    3 7


But it does apply to arrays:


    from numpy import arange
    a = arange(9).reshape(3,3)
    b = a
    print a
    print b

    [[0 1 2]
     [3 4 5]
     [6 7 8]]
    [[0 1 2]
     [3 4 5]
     [6 7 8]]



    b[0:2,0:2] = -123
    print b

    [[-123 -123    2]
     [-123 -123    5]
     [   6    7    8]]



    print a

    [[-123 -123    2]
     [-123 -123    5]
     [   6    7    8]]


To make arrays same by value, but distinct in memory, make an explicit copy:


    a = arange(9).reshape(3,3)
    b = a.copy()
    print a
    print b

    [[0 1 2]
     [3 4 5]
     [6 7 8]]
    [[0 1 2]
     [3 4 5]
     [6 7 8]]


Now change `b`


    b[0:2,0:2] = -123
    print a
    print b

    [[0 1 2]
     [3 4 5]
     [6 7 8]]
    [[-123 -123    2]
     [-123 -123    5]
     [   6    7    8]]


`a` was not touched.
Careful with array slicing:


    a = arange(9).reshape(3,3)
    b = a[:,:]   # is this slicing a copy?
    print a
    print b

    [[0 1 2]
     [3 4 5]
     [6 7 8]]
    [[0 1 2]
     [3 4 5]
     [6 7 8]]



    b[0:2,0:2] = -123
    print a
    print b

    [[-123 -123    2]
     [-123 -123    5]
     [   6    7    8]]
    [[-123 -123    2]
     [-123 -123    5]
     [   6    7    8]]


**Slicing arrays via `:` or via the Ellipsis object `...` creates a 'view' of
the original array, i.e. the new object still points to the same chunk of
memory.**

Apply an in-place operator, such as `+=`


    b += 3
    print a
    print b

    [[-120 -120    5]
     [-120 -120    8]
     [   9   10   11]]
    [[-120 -120    5]
     [-120 -120    8]
     [   9   10   11]]


It updated `b` in-place, i.e. used the same location in memory. And since `a`
still points to that same memory, it changes with `b`.

Now increment `b` with a normal `+` operator (not in-place):


    b = b + 20
    print a
    print b

    [[-120 -120    5]
     [-120 -120    8]
     [   9   10   11]]
    [[-100 -100   25]
     [-100 -100   28]
     [  29   30   31]]


An operator like `+` makes a copy of the array, and assigns the updated value to
that new location in memory. `a` and `b` now point to different locations in
memory.


    
