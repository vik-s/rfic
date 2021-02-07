---
title: Scikit-RF Tutorial (4) - Frequency Slicing
date: 2021-01-17 22:30:15 +/-0800
categories: [Python, scikit-rf]
tags: [programming]
math: true
---

In the fourth post of this series, we will focus on reducing the network data to a smaller subset of frequencies.

For your reference, here is a list of articles in this series.
1. [Getting Started]({% post_url 2021-01-04-ScikitRF-Get-Started %})
2. [The Network() Class]({% post_url 2021-01-12-The-Network-Class %})
3. [Manipulating Networks]({% post_url 2021-01-19-Manipulating-Networks %})
4. [Frequency Slicing]({% post_url 2021-01-19-Frequency-Slicing %}) (this article)
5. [Plotting]({% post_url 2021-02-07-Plotting %})

Here is a use case of why you might why you might want to do something like this. When making network analyzer measurements, it is common practice to take a wider range of frequencies than necessary during measurement. Doing this avoids having to recalibrate the setup, and redo the measurement if wider frequency data is required in the future. During the analysis phase, you might only want to focus on a narrow range of frequencies, or even a spot frequency, that is relevant to the circuit operation. This is where the need to carve out a subset of frequencies from a dataset arises. To cover this and other scenarios, let's dive into frequency slicing of networks!

# Extracting Spot Frequencies

Let's start with a simple example. Assume you want to extract network data at a single frequency point, say 2GHz. You can create a new network object with just a single frequency point by passing it as a string argument to an existing network object, as follows.

```python
# extract network data at a single frequency
nw_1f = nw['2ghz']
print(nw_1f)
```

Output:

```
2-Port Network: 'ntwk1',  2.0-2.0 GHz, 1 pts, z0=[50.+0.j 50.+0.j]
```

You have created a new network object that has only 1 frequency point. The frequency units are case insenstive, so you can also use `GHz` or `Ghz` without errors.

Using the methods of manipulating network data discussed in the [last post]({% post_url 2021-01-19-Manipulating-Networks %}), you can quite simply extract a single float value corresponding to a network parameter at a given frequency. As an example, let us extract $$S_{21}$$ at a given frequency using this single-line command.

```python
# extract s21 at a single frequency
s21_1f = nw['2ghz'].s21.s_db.item()
print(s21_1f)
```

Output:

```
-0.7856314044096142
```

`nw['2ghz']` creates a network object at a given frequency. `s21` extracts a new network object at a given frequency that only contains $$S_{21}$$. But this is still a network object. Using `s_db` returns the $$1\times1\times1$$ numpy array containing the required network parameter in dB format. To convert it to a python scalar of type `float`, we use the `item()` method that gives us the desired result.

# Extracting Frequency Ranges

Similarly, if you want to extract a range of frequencies, you can specify it using a string such as `4-6ghz`.The result is a new network object with a smaller slice of frequencies.

```python
# extract network data over a frequency range
nw_f = nw['4-6ghz']
print(nw_f)
```

Output:

```
2-Port Network: 'ntwk1',  4.0-6.0 GHz, 21 pts, z0=[50.+0.j 50.+0.j]
```

You now have a new network object that has been reduced to only 21 frequency points that you specified between 4 and 6 GHz. You could also repeat the frequencies units when specifiying a range like `4ghz-6ghz`. Or use other frequency units such as `4000mhz` or `4000-6000mhz`. As of scikit-rf v0.16, mixing frequency units like `4000mhz-6ghz` would not work.

If no units are specified in the string, like `4-6`, then the frequency unit defined in the`Network().frequency.unit` property is used to slice the network.

```python
# frequency units specified in the network object
print(nw.frequency.unit)

# check if both freq definitions are equivalent
print(nw['4-6'] == nw['4-6Ghz'])
```

Output:

```
GHz
True
```

# Interpolation and Extrapolation

If a frequency point or range is specified that is not part of the original frequency dataset but is still within the range of available frequencies, then the s-parameter matrix is interpolated to return the value at the specified frequency. In the example below, `2.005 GHz` is not in the original s-parameter dataset. If the s-matrix at that frequency is accessed, the interpolated value is returned.

```python
# example of frequency interpolation
s1 = nw['2.005ghz'].s
print(s1)
```

Output:

```
[[[-0.0496265 -0.28207265j  0.85581999-0.31951895j]
  [ 0.85581999-0.31951895j -0.0430651 -0.22325425j]]
```

If the frequency point or range is beyond the lower or upper limit, the network object is truncated at the lower or upper frequency point, respectively. No extrapolation is performed. For instance, if we specify a frequency range such as `2-11ghz` on a dataset that has a maximum frequency of `10ghz`, then you will see that the dataset is truncated at the highest available frequency.

```python
# example of frequency trucation
s2 = nw['2-11ghz']
print(s2)
```

Output:

```
2-Port Network: 'ntwk1',  2.0-10.0 GHz, 81 pts, z0=[50.+0.j 50.+0.j]
```

# Using Numpy Arrays

So far we have created slices in the frequency domain to create network objects. If you are already working with numpy arrays, then frequency slicing is simply a matter of matrix manipulation of numpy arrays. If you recall, a $$2\times2$$ s-parameter matrix with N frequencies is of size $$N\times2\times2$$, and numpy arrays are zero-indexed. If you need to access a spot frequency or a range of frequencies, you need to specify them as indices of the numpy array.

Let's use our earlier example of spot frequency data at 2GHz. From inspection of the frequency array `nw.f`, we know beforehand that element `10` is the frequency point corresponding to 2GHz, and we can access that directly as `nw.s[10,:,:]` to get the s-parameter matrix at 2GHz. A more useful representation would be to dynamically find that index without explicitly defining the index number by comparing the frequency array to the required value using `nw.f == 2E9`. This results in an array of boolean values that state if the condition has been met or not. This boolean array can be used in place of the frequency index to find the s-parameter matrix at that frequency.

```python
# extract spot frequency data using numpy array
nwk_1f = nw.s[nw.f == 2E9,:,:]
```

If you would like to access a range of frequencies using numpy arrays, you can use `logical_and()` to return a boolean array that returns `True` for every frequency that meets the speficied conditions, and `False` if it does not. In the example below, we extract a subset of frequencies between 4 and 6 GHz.

```python
# extract frequency range using numpy array
import numpy as np
nwk_f = nw.s[np.logical_and(nw.f >= 4E9, nw.f <= 6E9),:,:]
```

This gives you same network data in the form of numpy arrays as when you use `nw[4-6ghz].s`.

Apart from these methods already shown, you can use a variety of notations that are generic ways to slice numpy arrays. Some of these methods are  shown below.

```python
# generic frequency slicing using numpy arrays
s21 = nw.s[:21,1,0] # accesses the first 20 frequency points
s21 = nw.s[21:,1,0] # ignores the first 20 frequency points
s21 = nw.s[10:21,1,0] # access frequency points between 10 and 20
```

That covers the various ways of controlling the frequency content of networks in scikit-rf! In the next post, we will look at plotting network data in various projections such as rectangular, polar and smith, and also look at exporting network data to other formats.
