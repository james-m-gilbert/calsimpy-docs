---
title: CalSimPy
date: 2026-04-23
authors:
  - name: James M. Gilbert
    affiliations:
      - University of California, Santa Cruz
---

## Some Important Background Information

The CalSimPy package comprises a collection of scripts and functions that I built in the process of various CalSim-related analyses over the years. 
I decided to put them in a package to make maintanence, usage, and sharing easier.
I know CalSim modelers can be an opinionated bunch, so I want to be clear that my intent in sharing is not to promote or imply that my approach is "right" or better than other options.
In fact, the only thing I can confidently guarantee is that **CalSimPy** is *not* universally suitable for any and all CalSim analyses, applications, or use-cases.

Rather, I am making available the tools that I have found useful in case some modeler out their might also find them helpful. 
By publishing this code, I am also hoping to build on on recent efforts to make CalSim models and modeling tools more open and available.
I have benefitted in the past from the efforts of others in the CalSim modeling world, so it only makes sense that I contribute back what I can.

I would also like to emphasize that I assembled all of this over years not for the purpose of simply saying I had written the code, but to assist in pursuit of broader aims (CalSim-related analyses of various sorts).
That is to say, this code came about as a means to an end, not an end in and of itself.


I'm not a trained software engineer or a professional developer.
Many parts of the code were assembled at different times for different purposes, often on a short timeline.
That means the code is ugly and probably a bit clunky.
And the structure and purpose may not align with how you see things
or your particular needs.


That's fine - this was never meant to be a universal solution.
Hopefully, though, this and related efforts from the talented CalSim modeling community can serve as a reference point and catalyst for building something better.

## Introduction to Basic CalSimPy Concepts

The **CalSimPy** package is organized around modules that are targeted at specific models in the CalSim cinematic universe, including:

    - CalSim3
    - CalSimHydro
    - The coupled C2VSim DLL

Each of these modules consists of at least one object or class that serves as the main interface and "container" for model information.
There may be subclasses accessible from the main class (e.g. the CalSim3 class has associated *SV*, *DV*, and *Table* objects), and a set of functions or methods that do some action (usually retrieving, setting, or manipulating data).

The goal for each component is to be able to interact with models the same way the numerical engines do - through configuration or input files.
Where possible, data that is read in from files is stored in common Python data structures like lists, dictionaries, and Pandas DataFrames.
Most time series data (such a that stored in DSS files) will be converted to DataFrames for easier interchange with CSV files or other Pandas-compatible formats. 
For data that has a geographic component, like the C2VSim components, information is stored in dictionaries or GeoDataFrames.

```{note} A note about DSS file access

Access to DSS data in CalSimPy is currently supported through the integration of the `yapydss` (Yet Another Python DSS) package [GitHub yapydss](https://github.com/james-m-gilbert/yapydss).
This should be installed automatically when you set up **CalSimPy**, so there isn't really anything the casual user needs to know to get started.

This particular Python-DSS wrapper was built around the Excel add-in DLL back when the heclib source code was a closely guarded secret. 
It does some things well, some others not as well. 
It has worked for me pretty well over the last 9 years, but I don't pretend to think it cannot be improved upon.

```


## Installing the Package


I developed **CalSimPy** in a Python version 3.10.x environment.
I have not tested with other Python versions - for the best luck in getting everything to work, I'd start with a v3.10.x version.

```{hint} Use a virtual environment
I would highly recommend setting up a virtual environment for installing this package.
It takes a little extra work now, but will save you compatibility headaches down the road. 
```

```{important} Windows compatibility only!
While the Python code to interface with text files and other data formats should be broadly compatible across platforms, the DSS library around which **_CalSimPy_** is built currently only supports Windows.
There are ways to get around this if you feel adventerous, but that's more than I want to discuss here.

```

The code is hosted on [GitHub](https://github.com/james-m-gilbert/calsimpy), so you can clone, load, and develop your own version if you prefer that sort of thing.

You can also install it using the `pip` package manager using the following command in your favorite console or command line interface (remember to activate the virtual environment you intend to install the package in!) :

`pip install git+https://github.com/james-m-gilbert/calsimpy.git@v0.1.0-alpha`

This command will try to install the initial release version (v0.1.0-alpha, from 2026-04-27) available on GitHub.
The process should first download (clone) the repository, then work through the dependencies (more downloading involved), and finally install everything.

If it goes well, you'll see a bunch of text scroll by on the command window, with a message eventually saying something like *Successfully built calsimpy yapydss* and *Successfully installed ...* followed by a listing of all the installed packages and dependencies.
At this point, you are ready to start working with the package. 

**One more note** - If you work with an IDE like Spyder or VS Code, you will also need to install the corresponding kernels so you can connect through those programs. 
For example, you may need to run `pip install ipykernel` in your virtual environment to install an IPython interpreter that you can connect to VS Code.


```{important}
I have completed several test installations of the distribution package in virtual environments on my local machine. 
Although it works on my computer, I cannot guarantee it will work for anyone else "out of the box".
There is a chance that there may be some missing dependencies or package conflicts.
You will quickly find out if a package is missing if you get an error when trying to load CalSimPy modules in your interpreter.

```

## Organization of this Documentation

The rest of this documentation site is devoted to more detailed explanation of the **CalSimPy** functionality. 
It is organized into sections - one for each major component of the package:

- the CalSim (`cs3`) class
- the CalSimHydro (`csHydro`) class
- the CalSim groundwater (`cs3gw`) class
- and CalSim plots (`csPlots`) class

### Some final notes...

All explanations and examples are meant to be runnable by the modeler on their local computers.
The code examples provided assume that the user knows how to start a Python interpreter associated with the correct environment and to execute code via that interpreter.
This is often done through an IDE like VS Code or Spyder or a notebook environment like a Jupyter notebook.
References to results will encompass a range of formats, including statements printed to the console (or interpreter) window, variable states or characteristics, generated plots or figures, and data written to files (e.g. CSV or DSS).
Before proceeding, you should feel comfortable doing all of the above in your selected coding environment.

This is a work in progress - I will start with the basic package and functionality and add more documentation as time and funding support allows. 