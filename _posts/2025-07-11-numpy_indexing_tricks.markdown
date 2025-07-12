---
layout: post
title:  "NumPy Indexing Tricks"
date:   2025-07-10 14:42:47 +0000
categories: python numpy
---

# Numpy Indexing Tricks

> *C is a razor sharp tool, with which one can create an elegant and efficient program or a bloody mess.*
>     - *Brian Kernigan*

The brilliant quote above from Brian Kernigan (A developer of the early unix operating system) was of course talking about the C programming language, but I think the same could equally be said about NumPy. Without proper care, its all to easy to introduce subtle bugs into NumPy code that can be a headache to debug. These bugs often revolve around improper indexing. Despite these difficulties I think NumPy is an excellant and underused tool so I thought I would provide a few tips in this article to help ensure you end up with an elegant and efficient program, not a bloody mess.

## What is NumPy

For those unfamiliar with NumPy, it is a python library that provides multidimensional array data structures and routines for manipulating them. These data structures and routines are implemented in highly optimized C or fortran code. This makes them very fast, and also avoids much of the memory overhead that is associated with python objects and function calls. Due to this speed and efficiency its often preferable (or even necessary) to implement heavy duty numeric computation in numpy rather than pure python. It has the added nicety that it interoperates beatifully with the rest of the python data ecosystem (pandas, matplotlib, scipy, numba). I've shown an example of some numpy code below, which computes the pairwise euclidean distance between points in 3D space.


```python
%load_ext memory_profiler
```


```python
import numpy as np

def pdistnp(points1, points2):
    return np.sqrt(((points1[:,None] - points2)**2).sum(axis=2))
```

We can time how long it takes to run using the IPython cell magic command %%timeit


```python
rng = np.random.default_rng()

points1np = rng.normal(size=(100,3))
points2np = rng.normal(size=(100,3))
```


```python
%timeit pdistnp(points1np, points2np)
```

    179 μs ± 945 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)


It clocks in at ~200 μs. Lets implement the same functionality in pure python and see how it performs.


```python
from math import dist
from operator import sub

def pdistpy(points1, points2):
    return [
        [dist(p,q) for q in points2]
        for p in points1
    ]
```


```python
points1py = points1np.tolist()
points2py = points2np.tolist()
```


```python
%timeit pdistpy(points1py, points2py)
```

    712 μs ± 4.09 μs per loop (mean ± std. dev. of 7 runs, 1,000 loops each)


That clocked in at ~800 μs, so 4x slower than the numpy version! And that is speed difference is comparatively small compared to the kinds of speed ups it is typical to see when porting pure python code to NumPy. It is not unheard of to acheive 100-1000x speedups in some cases.

We can double check that they are producing the same results.


```python
np.allclose(pdistpy(points1py, points2py), pdistnp(points1np, points2np))
```




    True



So NumPy is faster, what about memory?


```python
import sys
```


```python
distmatnp = pdistnp(points1np, points2np)
```

The distance matrix is of size 100 x 100, so it has 10000 elements. Each element is a 64 bit floating point number, and so takes up 8 bytes. Multiplying these numbers together tells us that our array takes up 80000 bytes or 80KB.


```python
f"{distmatnp.size*distmatnp.itemsize*1e-3} KB"
```




    '80.0 KB'



A python float on the other hand takes up 24 bytes by default. Multiplying by the same number of elements gives us a total of 240000 bytes, or 240KB. So 3x larger than the NumPy array output.


```python
distmatpy = pdistpy(points1py, points2py)
```


```python
sys.getsizeof(1.0) * 100**2
```




    240000



So NumPy is faster **and** more memory efficient! Whats not to love? The answer is multidimensional...

## NumPy Vectorization

The core concept that you need to be comfortable with to write good numpy code is **vectorization**. This means expressing operations on *arrays* rather than looping over individual elements to apply an operation. For example, the snippet below subtracts two sets of numbers without any explicit looping. The operation is applied to the two arrays *elementwise*.


```python
np.array([1, 2, 3, 4]) + np.array([1, 2, 3, 4])
```




    array([2, 4, 6, 8])



In pure python, you would need to loop over the individual elements of each array and apply the operation to the elements directly.


```python
[x + y for x, y  in zip([1, 2, 3, 4], [1, 2, 3, 4])]
```




    [2, 4, 6, 8]



The above example is simple because we are working with 2 1D arrays. Things often get more complicated when we are working with higher dimensional arrays, and wanting to apply operations only over particular *axis* (dimensions). In the first example I showed, you may have noticed this syntax `(points1[:,None]`. If you are not familiar with this, it is an example of *array reshaping*. In order to understand why we need to do this, lets try skipping it and seeing what happens.


```python
def pdistnp2(points1, points2):
    return np.sqrt(((points1 - points2)**2).sum(-1))
```


```python
pdistnp2(points1np, points2np).shape
```




    (100,)



We were expected the result of the operation to be a 2D array (100 x 100), but what we got was a 1D array with 100 elements. What happened is NumPy applied the subtraction *element-wise*, meaning the first element of points1 was compared with the first element of points2, the second element with the second etc. What we want is to subtract *every element* of points2 from *every element* of points1.  In order to get what we want, we need to insert a *unit axis* (an axis of size one) in the points1 array.


```python
points2np[:,None].shape
```




    (100, 1, 3)



Now, when we subtract points2, which is a 2D array with shape 100 x 3, NumPy will do what is called *broadcasting*, which essentially means copying the elements from an axis of one of the arrays, to match the size of the same axis in the other. The rules for broadcasting can be found [here](https://numpy.org/devdocs//user/basics.broadcasting.html). In our case, what happens under the hood is shown below.


```python
# A      (4d array):  100 x 1   x 3
# B      (3d array):        100 x 3
# Result (4d array):  100 x 100 x 3
```

Step-by-step breakdown
1. NumPy first aligns the two arrays along the last axis
2. It finds that axis 1 in array A is of size 1 and axis 1 in array B is of size 100, so it 'copies' axis 1 of of A 100 times to match the size of B's axis 1
3. It finds that axis 0 of B is non-existant, so it inserts a unit axis and repeats the above step in reverse (i.e. copying the 0th element of B 100 times to match the size of the 0th axis of A)

Note: I put 'copying' in quotes because what goes on under the hood is slightly more complex than this, but its a good working mental model.

In practice this means that every element of points2 is subtracted from every element of points1, resulting in a 100 x 100 x 3 array of diffs. We then square and sum these diffs *across the last axis* to get the euclidean distance. By passing the `axis=2` argument to sum, we are telling it to sum across the x y z axis. If we omitted this argument, we would find that the result returned would be a scalar value, as by default the sum is computed over all elements. With the `axis=2` argument however, we end up with a 100 x 100 array of pairwise diffs, just what we want!

## Scaling up

Often when developing NumPy code (or other array based code such as pytorch), you will start of with data that has fewer axes. This is because working with smaller data is usually faster, and in the early stages of development you want to prioritize iteration speed so you can try lots of things quickly and see what works. This is common in machine learning for example, where you may do some experimental work with grayscale images before jumping up to full color images, or where you may work with a single instance before moving on to processing batches of images. What you want, is to seamlessly pass in your scaled up data once you have worked out your algorithm, and for everything to work in the same way.

Unless you have taken care to enable this though, you will run into issues. Lets see how by scaling up our distance calculations to working with batches of 3D points as opposed to single instances. We will try to compute the pairwise distances between pairs of 3D points.


```python
batched_points1np = np.tile(points1np[None], (10, 1, 1))
batched_points2np = np.tile(points2np[None], (10, 1, 1))
```

We have inserted a unit axis before the 0th axis of our points and copied it 10 times to simulate a batch of 3D points rather than a single instance. We can check this by looking at the shape of the arrays.


```python
(batched_points1np.shape, batched_points2np.shape)
```




    ((10, 100, 3), (10, 100, 3))



Now lets try to apply the functions we have already written to do the same thing on our scaled up dataset.


```python
batched_pdists = pdistnp(batched_points1np, batched_points2np)
```


```python
batched_pdists.shape
```




    (10, 10, 3)



Looking at the shape of the arrays we can clearly see that something has gone wrong. We were expecting out output array of shape 10 x 100 x 100 representing the pairwise distances between each pair of 3D points in our batch. What we got out was an array of size 10 x 10 x 3. So what went wrong? Lets step through our code step by step to find out.

Note: This is why developing complex numeric code in notebooks is such a good idea. Its easier to spot and debug issues like this!


```python
(batched_points1np[:,None] - batched_points2np).shape
```




    (10, 10, 100, 3)



Previosly this step resulted in an array of shape 100 x 100 x 3. Here we are getting an array of 10 x 10 x 100 x 3. Lets check the output of our reshaping operation.


```python
batched_points1np.shape, batched_points1np[:,None].shape
```




    ((10, 100, 3), (10, 1, 100, 3))



We can now see the problem, we wanted to insert a unit axis just before the final x y z axis. Instead, we inserted one just after the batch axis. This is because the `array[None]` syntax will insert axis *from the left*. So `array[None]` will insert a unit axis before all other axes, `array[:,None]` will insert an axis *after* the first axis, etc. In our code, the `array[:,None]` inserted a unit axis *after the first axis* rather then *before the final axis*, which is what we wanted. What we want to do, is insert a unit axis *before* the final axis. We can do you using the ... notation, i.e. `array[...,None]`. What this does is make numpy insert unit axes *from the right* rather than the left. So `array[...,None]` would produce an array of shape N x 1, with N being the original shape of the array, as opposed to 1 x N. We can alter it so that it inserts a unit axis just before the final axis like so.


```python
batched_points1np[...,None,:].shape
```




    (10, 100, 1, 3)



If we try our original broadcasting operation though, we will get an error from numpy telling us that the arrays cannot be broadcast together. I encourage you to try and figure out why this is before reading the explanation.


```python
batched_points1np[...,None,:] - batched_points2np
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Cell In[27], line 1
    ----> 1 batched_points1np[...,None,:] - batched_points2np


    ValueError: operands could not be broadcast together with shapes (10,100,1,3) (10,100,3)



```python
# A      (4d array):  10 x 100 x 1   x 3
# B      (3d array):      !10 cannot be broadcast to 100
# Result (4d array):       10  x 100 x 3
```

As it says in the docs, after the axis have been aligned on the left the arrays can be broadcast iff for each pair of axes:
1) They are the same size
2) One of them is of size 1
3) One of them is of size 0

In the above example the 0th axis of the batched_points2np array (size 10) is aligned with the 1st axis of the batched_points1np array (size 100). As neither of these have size 0 or 1, and they are not the same size, they cannot be broadcast.

We can fix this by inserting a unit axis in-between the 0th and the 1st axis like so.


```python
batched_points2np[...,None,:,:].shape
```




    (10, 1, 100, 3)



This reads as "consume as many axis as you can (...), insert a unit axis, then take the following two axes as is." This has the effect of inserted a unit axis before the 2nd axis *from the right.*

Now, when we subtract `batched_points2np` from `batched_points1np`, we get an output that is of the right shape, i.e. 10 x 100 x 100 x 3


```python
(batched_points1np[...,None,:] - batched_points2np[...,None,:,:]).shape
```




    (10, 100, 100, 3)



The nice thing about writing the code this way, is that it is robust to changes in the number of dimensions in our input arrays. If we wanted to go back down to working with single instances of 3D points rather than batches, we do not need to change anything. We still end up with correctly sized output arrays, despite using the exact same indexing logic.


```python
(points1np[...,None,:] - points2np[...,None,:,:]).shape
```




    (100, 100, 3)



Lets write a new function with the updated indexing logic, and check that it returns consistent results across inputs with different sized inputs.


```python
def pdistnp3(points1, points2):
    return np.sqrt(((points1[...,None,:] - points2[...,None,:,:])**2).sum(axis=2))
```


```python
pdistnp3(batched_points1np, batched_points2np).shape
```




    (10, 100, 3)




```python
pdistnp3(points1np, points2np).shape
```




    (100, 100)



The shapes of the output arrays show that our solution is not correct. We should expect to see a 10 x 100 x 100 array in the batched case, and a 100 x 100 array in the single instance case. The problem lies in the `.sum(axis=2)` which is similarly not robust to changing axis. We can see what happens by again breaking down the computation into two steps an checking the intermediate shapes.


```python
sqrdiffs = ((batched_points1np[...,None,:] - batched_points2np[...,None,:,:])**2);sqrdiffs.shape
```




    (10, 100, 100, 3)




```python
sqrdiffs.sum(2).shape
```




    (10, 100, 3)



The output makes it clear that we have summed along the wrong dimension. In the original case with only 3 axis, the 2nd axis was the last axis (the x y z axis), however, since we added the batch axis, the 2nd axis now represents the enumeration of points rather than the x y z axis. We can fix this by observing that the x y z axis is *always* the last axis in the array, regardless of wether we add a batch axis or not. We can explicitly encode this observation by specifying that the sum should be computed over the last axis; `.sum(axis=-1)`.


```python
def pdistnp4(points1, points2):
    return np.sqrt(((points1[...,None,:] - points2[...,None,:,:])**2).sum(axis=-1))
```


```python
pdistnp4(batched_points1np, batched_points2np).shape
```




    (10, 100, 100)




```python
pdistnp4(points1np, points2np).shape
```




    (100, 100)



The output is now the correct shape regardless of the number of dimensions in the input array. We can also check that they are producing the same values.


```python
np.allclose(pdistnp4(batched_points1np, batched_points2np)[0], pdistnp4(points1np, points2np))
```




    True



With these changes we can now iterate fast developing algorithms on single instances, or scaled down versions of our data, and then effortlessly apply the same algorithms to the scaled up version. Without doing this, you can end up introducing insidious bugs that can be hard to track down, as they may silently produce incorrect results without raising explicit errors.

## Summary

By thinking ahead of time about how the shape of your data may change, you can write code to anticipate these changes, speeding up your development cycle and avoiding nasty bugs. A key trick is to think about whether your axes indices are invariant from the right or the left. i.e if I insert a dimension I plan to introduce later, will the index of this axis change? Will it also change if I index it from the opposite end? i.e. `array[-1]` instead of `array[2]`. Broadcasting is the killer feature of NumPy, which will allow you to write faster more efficient code, but it should be treated with respect. If you find you struggle with this, don't worry; its complicated! My recommendation is, find a toy problem, fire up a jupyter notebook and mess around with some arrays.

As always feel free to reach out if you like the post and want to know more. Happy NumPy'ing!
