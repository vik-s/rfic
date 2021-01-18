---
title: Scikit-RF Tutorial (2) - The Network() Class
date: 2021-01-09 20:40:20 +/-0800
categories: [Python, scikit-rf]
tags: [programming]
math: true
---

In this article, we will look into the fundamentals of the `Network()` class. It is a fundamental building block of the scikit-rf package, and we will use it repeatedly to wrangle rf network data. Learning to work with objects derived from this class will go a long way in getting proficient with the use of this python package.

For your reference, here is a list of articles in this series.
1. [Getting Started]({% post_url 2021-01-04-ScikitRF-Get-Started %})
2. [The Network() Class]({% post_url 2021-01-12-The-Network-Class %}) (this article)
3. [Manipulating Networks]({% post_url 2021-01-19-Manipulating-Networks %})
4. [Frequency Slicing]({% post_url 2021-01-19-Frequency-Slicing %})

For the uninitiated, a [class](https://en.wikipedia.org/wiki/Class_(computer_programming)) is a template which contains pre-defined data structures, initial values and functions or methods defining a sequence of programming operations. In scikit-rf, the `Network()` class is a collection of attributes and functions available for handling RF network data. To perform data analysis and manipulations on networks, we must first create a network object, which is an instance of the `Network()` class.

The construction of a network object depends on where the network s-parameter data is available to begin with. Network data can be found in several file formats such as the generalized [Touchstone](http://www.ibis.org/touchstone_ver2.0/touchstone_ver2_0.pdf) format, Pathwave (formerly Keysight) specific formats such as MDIF/CITI (Advanced Design System), or MDL/MDM (ICCAP), MAT files from Matlab, or simply comma-separated values CSV files. Scikit-rf supports direct import of touchstone files into a network object data structure. For other formats, you will need to cast the data into [numpy](https://numpy.org/) arrays or touchstone files before building the network object. Therefore, we will also look at the creation of network objects from numpy arrays.

# Network Objects from Touchstone

This is the most straightforward way to create a network object. Touchstone data is commonly available from RF simulation software or from vector network analyser (VNA) measurements. Here is an example that loads the touchstone file into a network object:

```python
# get the example data file path in skrf
# you can add a path to your own file instead
import skrf as rf
import os
skrfpath = os.path.dirname(rf.__file__)
datafile = os.path.join(skrfpath, 'data', 'ntwk1.s2p')

# create a network object with
# rf.Network(/path/to/filename.s2p)
nw = rf.Network(datafile)

# print it
print(nw)
```

Output:

```
2-Port Network: 'ntwk1',  1.0-10.0 GHz, 91 pts, z0=[50.+0.j 50.+0.j]
```

The path to the n-port touchstone file (*.snp) is passed as an argument in the creation of the network object `nw`. In this example, we use an example touchstone file `ntwk1.s2p` included with scikit-rf package, and so the first few lines of code are meant to access that. If you have an s-parameter file in the same path as your jupyter-notebook is saved, you can just pass the `filename.sNp` file as an argument. We are using two-port networks here for simplicity; you can load a file with more ports if you like. Each network object has a name (`ntwk1` in this case) associated with it for convenient identification of the dataset.

# Network Objects from numpy Arrays

If you already have the network data available to you in the form of numpy arrays, either from computations, csv files, or custom parsing routines, you can use it to create a network object. If you have data in csv format, you can use the built-in [loadtxt()](https://numpy.org/doc/stable/reference/generated/numpy.loadtxt.html) or [genfromtxt()](https://numpy.org/doc/stable/reference/generated/numpy.genfromtxt.html#numpy.genfromtxt) methods in numpy to load comma separated values into numpy arrays.

Let us build a two-port network from numpy arrays. For this network, we have complex-valued numpy arrays for each of the s-parameters `s11`, `s12`, `s21` and `s22`. For the sake of this example, these are randomly generated complex numbers. Each of these quantities are of length N, corresponding to N frequency points `f`, expressed in Hertz. Here, we generate `N=91` frequency points equally spaced between 1 and 10 GHz, and corresponding s-parameters of the same length. The steps to create a network object from this information are:

- Create a $$ 2\times2 $$ s-parameter matrix `si` at a given frequency `i`
- For each frequency, stack the s-parameter matrix along the third dimension at create a $$2\times2\times N$$ matrix
- Reshape the s-parameter matrix into an $$N\times2\times2$$ matrix
- Create the network object by converting frequency into GHz units

Python code to implement these steps is as follows.

```python
# constructing a network object from numpy arrays
import numpy as np

# 91 points of random complex numbers for s-parameters
s11 = s12 = s21 = s22 = np.random.rand(91)+1j*np.random.rand(91)
# 91 frequency points between 1 and 10 GHz
f = np.linspace(1,10,91)*1e9

# define a 2x2 s-matrix at a given frequency
def si(i):
    ''' s-matrix at frequency i'''
    return np.array(([s11[i],s12[i]],
                     [s21[i],s22[i]]),
                    dtype='complex')

# number of frequency points
N = np.size(f)

# stack matrices along 3rd axis to create (2x2xN) array
s = si(0)
for i in range(1, N):
    s = np.dstack([s,si(i)])

# re-shape into (Nx2x2)
s = np.swapaxes(s,0,2)

# create network object with frequency converted to GHz units
nw2 = rf.Network(name='nw_from_numpy',s=s,frequency=f*1e-9, z0=50)
print(nw2)
```

Output:

```
2-Port Network: 'nw_from_numpy',  1.0-10.0 GHz, 91 pts, z0=[50.+0.j 50.+0.j]
```

Thats it! You now have network objects you can work with. The network object has frequency information, network data and reference impedances for each port, along with a wide variety of calculation and plotting functions that we will get into shortly. For now it suffices to say that the `Network()` class internally handles the data parsed from the touchstone file as complex numpy arrays.

Frequency units and reference impedances are important when defining network objects and naming network objects is useful when generating plot labels. Let us take a look at these topics  next.

# Frequency Units

In the network object constructions so far, notice that we did not have to worry about frequency units when building the network object from a touchstone file. This is because the frequency units are automatically set from the touchstone header file which defines it. However, when we built a network object from numpy arrays, we needed to convert it to GHz units because frequency units have a default value of GHz unless specified otherwise.

In our example with numpy arrays, we know that the frequency array `f` goes from 1 to 10 GHz with 91 steps in between. We can explicitly create an instance of the `Frequency()` class - a frequency object `f_obj`  - with this information, as shown below.

```python
# manually create frequency object
f_obj = rf.Frequency(1, 10, 91, 'ghz')
```

In many cases this is not a practical way to generate a frequency object. When the frequencies are unknown or the frequency spacing is uneven, you can construct a frequency object using the `from_f()` function, where we specify the frequency array `f` and the units of the array `hz` (remember that the original frequency units were Hertz) as the arguments of the function.

Finally, during the construction of the network object, the frequency object `f_obj` can be used as the frequency argument instead of the numpy array `f` as shown earlier.

```python
# create frequency object from numpy array
f_obj = rf.Frequency.from_f(f, unit='hz')

# create network object using frequency object
nw3 = rf.Network(name='nw_from_fobj',s=s,frequency=f_obj, z0=50)

# print it
print(nw3)
```

Output:

```
2-Port Network: 'nw_from_fobj',  1000000000.0-10000000000.0 Hz, 91 pts, z0=[50.+0.j 50.+0.j]
```

Notice that the units in the network object we just created are Hz, just like we defined in the frequency object. This is in contrast to the GHz units of the network object created from the touchstone file example.

# Network Object Naming

When we created the network object from a touchstone file, its name is assigned to be the name of the touchstone file. When creating a network object from numpy arrays, we specificed the name using the `name` argument. If you need to change the name of any object, you simply assign a new string value to the `name` attribute of the network object, like this:

```python
# changing object name
nw3.name = 'new_name'

# print it
print(nw3)
```

Output:

```
2-Port Network: 'new_name',  1000000000.0-10000000000.0 Hz, 91 pts, z0=[50.+0.j 50.+0.j]
```
Assigning names to network objects is helpful because plot labels are automatically generated for data plotted from these network objects. We will see this when we delve into plotting with scikit-rf.

# Reference Impedances

The argument `z0` passed during network object creation is the reference impedance for s-parameters. When a single number is specified, all ports are in the N-port network as assigned the same reference impedance `z0`. There may be occasions where one or several ports have different termination impedances. To specify different reference impedances at each port, `z0` can also be a list of impedances corresponding to each port as `z0=[z01, z01, ...,  z0N]`. Each reference impedance can be a real or a complex numner.

This ends the second part of this tutorial series! We created basic network objects from a few different data sources, and learned how to handle frequency units, naming and reference impedances. These network objects serve as building blocks for the next tutorial where we will discuss manipulation of network data using these objects.
