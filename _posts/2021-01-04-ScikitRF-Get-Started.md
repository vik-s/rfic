---
title: Scikit-RF Tutorial (1) - Getting Started
date: 2021-01-04 19:39:10 +/-0800
categories: [Python, scikit-rf]
tags: [programming]
pin: true
---

Welcome! This post is the first of a series of blog posts on RF and microwave analysis using python. At first, I thought I will design this to be a course with a limited number of modules, but as I wrote the content, I decided to adapt into a running series that I can keep adding to. The field of RF and microwaves is vast and it is better not to set boundaries. It will also keep me motivated to learn new things and add them to this tutorial series.

As you go through this series, you will learn how to manipulate, analyze, export and plot network data for starters. You will perform calibrations and de-embedding on network data, and learn to extract equivalent circuit values of common rfic components. Since the topics are open-ended, please feel free to contact me with your suggestions, and I will do my best to learn and write about it.

For your reference, here is a list of articles in this series.
1. [Getting Started]({% post_url 2021-01-04-ScikitRF-Get-Started %}) (this article)
2. [The Network() Class]({% post_url 2021-01-12-The-Network-Class %})
3. [Manipulating Networks]({% post_url 2021-01-19-Manipulating-Networks %})
4. [Frequency Slicing]({% post_url 2021-01-19-Frequency-Slicing %})

We won't get into the basics of python here, since there are many wonderful books, youtube channels and blogs that teach much more than we can cover in this blog. But don't worry, if you're just starting out with python, you will still be able to follow along. The python package we will deal with in this series is [Scikit-RF](https://scikit-rf-web.readthedocs.io/), which is an open source project hosted on [Github](https://github.com/scikit-rf/scikit-rf) and is the most complete python package available today for RF analysis. So lets get into it, and setup the necessary software to start this journey!



# Installing Python

For this tutorial, we will install the [Anaconda](https://www.anaconda.com/products/individual) distribution for python because it provides a comprehensive set of packages that are suitable for science and engineering. If you have no prior python experience, installing Anaconda will give you a sufficiently complete python installation that should meet most of your scientific needs. The installer is fairly large in size (~450MB) and is available for Windows, Mac and Linux. If you prefer a leaner installation without all the packages installed, you can use the [Miniconda](https://docs.conda.io/en/latest/miniconda.html) installation instead. This allows you to only install the packages you require because it is likely that you will not use most of the packages installed in the Anaconda distribution. You can also use the python installations that ship with Linux and MacOS distributions if you install the required packages, but we will not pursue this approach here.

# Code Editors

With an Anaconda installation, you have several programs you can use to write and execute python code. They are:

- Jupyter Notebook - A code editor that runs in a web browser
- Spyder - A full featured development environment with an IPython console
- Prompt - A command line interface to write and execute programs

You can read the [documentation](https://docs.anaconda.com/anaconda/user-guide/getting-started/) on these editors if you have trouble getting started with any of them. For the sake of this tutorial, we will use a Jupyter notebook, although you could just as easily do the same with the Spyder IDE if you prefer.

# Virtual Environments

In python, you can create virtual environments which are just sandboxed areas where you install packages that are necessary for a project. This is better than installing packages globally (also called the `base` environment) because there could be scenarios where installing certain packages could break other projects that use the same environment due to dependancy conflicts. Dependancy conflicts is a fancy name for when project A requires package-X version 1, but project B requires package-X version 2 only, and not version 1. In this scenario, one of the projects is bound to break. For more on this subject, a good explanation of python environments is available in this [article](https://realpython.com/python-virtual-environments-a-primer/).

In the Anaconda distribution of python, we can use `conda` as the environment manager. Using `conda`, we will create and activate a new environment called `myprj`. Next, we will install scikit-rf and jupyter notebook using `conda install` within the `myprj` environment. The next section shows a systematic guide to get everything set up.

# Step-by-Step Guide

If you are on Linux or Mac, you can use a terminal window for the following steps. Powershell is a good choice for Windows. Regardless of operating system, you can also use the Anaconda prompt from the Anaconda distribution.

1. **Create a virtual environment**

    ```bash
    conda create -n myprj
    ```

2. **Activate your environment**

    ```bash
    conda activate myprj
    ```

3. **Install scikit-rf and jupyter notebook**

    ```bash
    conda install -c conda-forge  scikit-rf
    conda install jupyter
    ```

4. **Start and create a jupyter notebook**

    ```powershell
    jupyter-notebook
    ```

    Click on the drop-down box on the top-right corner and create a new python 3 notebook.

5. **Check scikit-rf installation**

    Type the following in a cell and run the notebook. You should see the current version of scikit-rf you have installed. That's it! Now you have everything needed for RF analysis!

    ```python
    import skrf as rf
    print(rf.__version__)
    ```

    Save your jupyter notebook. For the rest of this tutorial, we will exclude the import statement in code examples as it is assumed that the package is already loaded. The convention of importing it as `rf` is purely arbitrary; any other choice of name is equally valid. We will continue to add cells to this notebook and explore the various features of scikit-rf in the next article!

