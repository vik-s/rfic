---
title: Scikit-RF Tutorial (3) - Manipulating Networks
date: 2021-01-14 21:06:10 +/-0800
categories: [Python, scikit-rf]
tags: [programming]
math: true
---

In the [previous post]({% post_url 2021-01-12-The-Network-Class %}) in this series, we saw how network objects are created from various data sources. In the third article of this series, we look at extracting a subset of the network data, like $$S_{11}$$ or $$S_{21}$$, either as a network object or as a numpy array. Learning to manipulate network data is important for calculations that can be performed on the data later on, such as lumped element model extractions for transistors or inductors as an example. We will also look at conversion to other two-port network parameters such as Y and Z, and go deeper into handling data units as well.

For your reference, here is a list of articles in this series.
1. [Getting Started]({% post_url 2021-01-04-ScikitRF-Get-Started %})
2. [The Network() Class]({% post_url 2021-01-12-The-Network-Class %})
3. [Manipulating Networks]({% post_url 2021-01-19-Manipulating-Networks %}) (this article)
4. [Frequency Slicing]({% post_url 2021-01-19-Frequency-Slicing %})

# Manipulating Network Objects

Let us say that you would like to look at input return loss of a network, $$S_{11}$$. To extract 1-port s-parameters as network objects from an multi-port network object `nw`, you can simply access the corresponding network parameter as a property as shown below.

```python
# extract 1-port objects from multiport objects
s11 = nw.s11
print(s11)
```

Output:
```
1-Port Network: 'ntwk1',  1.0-10.0 GHz, 91 pts, z0=[50.+0.j]
```

Note that `s11` is another network object. Similarly, you can access other properies of the network such as `s21`, `s12` or `s22`. If you have a network that is greater than two ports, you can still access any arbitrary network parameter using this representation, which is rather handy.

If you prefer to use an index-like represention for the s-parameter network object, then you can access the individual network parameters using indices within square brackets. In this form, the index is referenced to one, to resemble the representation of s-parameter indices. For example, use indices of `[1,1]` if you want to get $$S_{11}$$. It is easy to verify that this method of 1-port network representation is the same as representing it as a property of the multi-port network as shown earlier. The choice of representation is simply a matter of coding style and personal preference.

```python
# 1-port objects using index representation
s11 = nw[1,1]
print(nw[1,1] == nw.s11)
```

Output:

```
True
```

So, why should we keep these 1-port networks as objects? The reason is to access the built-in methods of the `Network()` class that allow plotting, transformations and data export. We will look at some of these methods and attributes in this chapter, and discuss plotting as a separate topic later. You can take a quick look at the methods available to any object in python by listing them using `dir(nw)`, where `nw` is the network object whose available methods we want to view.

# Manipulating Numpy Arrays

Since python has a wide range of packages for plotting, statistics and data analysis that use numpy under the hood, you might prefer handling network data as numpy arrays. To do this, you can simply use the `s` property of the network object like `nw.s`. We already saw that the s-parameter data corresponding to a 2-port network was a $$N\times2\times2$$ matrix corresponding to N frequency points. In numpy, matrix data is zero-indexed, so accessing network parameter $$S_{11}$$ would require indices of `[0,0]`. Here is an example:

```python
# accessing network data as numpy arrays with zero-referenced indices
s11 = nw.s[:,0,0] # for S11
s21 = nw.s[:,1,0] # for S21
```

You might notice that `nw.s` gives you numpy arrays but `nw.s11` gives you a network object. If you already have 1-port network objects extracted from a multi-port network object, as described earlier, then you can convert them to numpy arrays as follows.

```python
# converting a 1-port network object into a numpy array
s11 = nw.s11.s.squeeze()

# or equivalently,
s11 = nw[1,1].s.squeeze()
```

Why did we need to use numpy's `squeeze()` method? Converting a 1-port network object into a numpy array using `nw.s11.s` results in a $$N\times1\times1$$ array, where N is the number of frequency points. The `squeeze()` method collapses the matrix dimensions to give a 1-dimensional numpy array of size N, corresponding to N frequency points.

```python
# check shape of 1-port s-parameter
print(np.shape(nw.s11.s))

# after squeeze()
print(np.shape(nw.s11.s.squeeze()))
```

Output:

```
(91, 1, 1)
(91,)
```

# Alternate 2-port Network Parameters

So far, we have only looked at s-parameter matrices. With scikit-rf, you can easily convert to other [two-port network parameters](http://en.wikipedia.org/wiki/Two-port_network) such Y, Z, H, T and ABCD matrices by accessing the appropriate property of the network object as shown below. The result is a numpy array of size $$N\times2\times2$$, for a 2-port network with N frequency points.

```python
# conversion to other network parameters
y = nw.y # admittance y-parameters
z = nw.z # impedance z-parameters
h = nw.h # hybrid h-parameters
t = nw.t # transfer t-parameters
a = nw.a # cascade ABCD-parameters
```

If you need to convert between parameters, you can use the conversion functions available from the scikit-rf module. For example,

```python
# conversion between network parameters
zn = rf.y2z(y) # convert y to z
sn = rf.z2s(zn, z0=50) # convert z to s, z0 = 50 ohm (default)
```

Although many commonly used conversion functions exist, some don't. For instance, `yn = rf.t2y(nw.t)` will result in a `NotImplementedError`. In such cases, you will need to  code your own conversion functions which you can [contribute](https://github.com/scikit-rf/scikit-rf/wiki/Development) to the project by creating a pull request.

# Handling Data Formats

Network data is stored as complex numbers in numpy arrays and you can easily convert it to magnitude-phase, real-imaginary or decibel representations using scikit-rf without having to manipulate data using numpy methods. For example, if you want $$S_{21}$$ in dB, access the `s_db` attribute of the network object and slice the numpy matrix using indices to get $$S_{21}$$. 

```python
# accessing network data in db units
s21_in_db = nw.s_db[:,1,0]
```

Alternatively, you can get the same result by extracting a 1-port network object `s21` and getting the `s_db` property from that object. We will have to `squeeze()` this numpy array to collapse the matrix into a linear array. Here is how that is done.

```python
# accessing network data in different units
s21_in_db = nw.s21.s_db.squeeze()
```

You can easily verify that the two representations show above give the same result. That is, $$S_{21}$$ in decibel units.

It is also simple to extract other network parameters similarly. Let's say you want to look at $$Y_{11}$$ which is often used in inductance extractions, it is straightforward to plot quality factor of an inductor using the `y_re` and `y_im` properties of the network object.

```python
# Q factor example
import numpy as np
Q = np.abs(nw.y_im[:,0,0]/nw.y_re[:,0,0])
```
`np.abs` takes the absolute value of a quantity using the function available in the numpy package.

To see the available attributes for handling data formats in scikit-rf, you can use the `dir()` function to list the directory of available methods in the network object, but narrowed down to only the ones that are relevant to s-parameters by filtering out those that start with `s_`. 

```python
# find attributes to handle data formats for s-parameters
attr = [f for f in dir(nw) if f.startswith('s_')]
print(attr)
```

Output:

```
['s_active', 's_arcl', 's_arcl_unwrap', 's_db', 's_db10', 's_def', 's_deg', 's_deg_unwrap', 's_im', 's_invert', 's_mag', 's_rad', 's_rad_unwrap', 's_re', 's_time', 's_time_db', 's_time_impulse', 's_time_mag', 's_time_step', 's_vswr']
```

Here you find other functions and attributes that you can use to handle your network data. For example, `s_vswr` directly plots your reflection coefficents as a VWSR (voltage standing wave ratio). You can plot the phase of your complex data in degrees (`s_deg`) or radians (`s_rad`), with the option to unwrap phase data beyond the -180$$^\circ$$ to 180$$^\circ$$ limits (`s_deg_unwrap`). You can also extract arclength (`s_arcl`), inverted s-parameters (`s_invert`), active s-parameters with a given port excitation (`s_active`) or a variety of time domain parameters (`s_time_*`).

You can similarly list out the directory of methods for other network parameters if required, like those starting with `y_` or `z_`.

You now have a grasp on wrangling network data to your own advantage! With all network parameters under your control in the units that you want, you are ready to perform computations on the network data, or simply plot them against a frequency axis. We looked at subsets of network data in this post, but in the next one, we will look at extracting a subset of the frequency data for network analysis.
