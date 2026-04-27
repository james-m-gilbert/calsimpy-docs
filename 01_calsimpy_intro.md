---
title: Introduction to CalSimPy Basics
date: 2026-04-27
authors:
  - id: jmg
---

## CalSimPy: The CalSim (`cs3`) object

The `cs3` object is the component of **_CalSimPy_** meant for use with CalSim model files.
The object includes containers for information (e.g. a CalSim study configuration or decision variable (DV) time series) as well as methods (functions) for interacting with that data.
The material on this page provides an introduction to these components using some basic illustrative examples.

To start working with a **_CalSimPy_** CalSim functionality, we need to load the package and then initialize a CalSim object.
We load the package through simple import statements (noting that the required DSS library will be loaded automatically when CalSimPy modules are loaded). 
We usually initialize a CalSim object by reading in a WRIMS *launch* or *config* file associated with a particular CalSim study.
The following code shows an example of how to do this:

```{code} python
:label: load-calsimpy01
:caption: Loading CalSimPy package and initializing object
:linenos: 

from calsimpy import cs3  #<-- loads the main CalSimPy class 'cs3'

# set path to a launch file for a study you want to load
cs3_launch_fp = r'../example_cs3_study/calsim3_example.launch' 

# let's initialize the study - the call to the 'cs3.calsim' function
# will return a calsim object - lets assign that to the variable `cs3_study`
cs3_study = cs3.calsim(launchFP = cs3_launch_fp, calsim_version=3, reorg=True)

cs3_study

```

The import statement on line 1 in the example code will hopefully make sense.

The next line sets a path to a launch file associated with a CalSim study we want to load. 
The path shown is an example for illustrative purposes - you will want to replace this with a path to a launch file on your own computer.

The code on line 8 is what initializes a CalSim object - it reads in the information in the provided launch file and stores that internally. 
We specify the path of the launch file using the `launchFP` argument.
You can also use `configFP` to set the path to a *__study.config* file if you prefer.

We provide two additional arguments beyond the launch file path to ensure the code knows where to look for things. 
These are in place because (believe it or not) I developed the first versions of this for working with both CalSimII and CalSim3 and before the folder structure was reorganized (i.e. the `CONV`/`common` folder structure was replaced). 
It's been on my list to make these the sensible defaults since CalSim work is nearly exclusively reorganized CalSim3 now.

* **calsim_version** = 3: This is provided to ensure the code knows it is a CalSim3 model
* **reorg = True** : This is provided to ensure the code knows the folder structure is the 'newer' reorganized form

The final statement - simply calling our new CalSim object `cs3_study` - should return (to the interpreter or console window) a listing of some of the information it just read in, similar to the example below.

```python

#######################################
CalSim object:
	Version: CalSim3
	Study Directory: D:/02_Projects/CalSim/proj/2023_CalClimateInit/calsim/CalSim3_Scenarios\s0002_DCR2023_9.3.1_danube_adj/Model_Files
	Main file: mainCS3_ReOrg_UWplusVF.wresl
	Configuration:
		 Start Year-Month-Day: 1921-10-31
		 End Year-Month-Day: 2021-9-30
		 DV Filename: DCR2023_DV_9.3.1_v2a_Danube_Adj_v1.8.dss
		 SV Filename: DCR2023_SV_Danube_Adj_v1.8.dss
		 DSS DV A Part: CALSIM
		 DSS F Part: L2020A
#######################################
```

If you got an error when trying to read in the *launch* (or *config*) file, check that the file paths provided are correct. 
If a CalSim study was copied in from a different location and the launch file contains absolute file paths, CalSimPy will not be able to find the files it's looking for when you try to initialize.
Another error might arise if you specify `reorg=False` and the study follows a reorganized structure - the code will look for SV and DV files in locations that don't exist.

If the CalSim object initialized successfully, you are ready to start accessing the functions and data through that object.
For more on that, continue below.

## Reading State Variable (SV) and Decision Variable (DV) Data

Retrieving, viewing, plotting, and interacting with time series data is one of the most common tasks you probably undertake as a CalSim modeler. 
The SV and DV classes associated with the CalSim object are meant to help with that process.
As a reminder, the DSS interface functionality in the current version of **_CalSimPy_** is provided through yapydss and will not have the same functionality as other Python-based DSS wrappers.

We can access SV or DV data through the `SVdata` or `DVdata` objects attached to our CalSim study variable (`cs3_study`).
Specifically, there are `getSVts` (get SV time series) and `getDVts`(get DV time series) functions that facilitate the filtering and retrieval of DSS data paths that match the provided criteria. 

These function calls take the form of 

    *{cs3Object}.[S,D]Vdata.get[S,D]Vts(filter=[{search terms for DSS variables}])* 
    
where the *{cs3Object}* is a previously-initialized CalSim object, *[S,D]* designate if a SV or DV data file should be referenced, and the search terms are one or more comma-separated text terms used to filter through the DSS path catalog. 

Some examples of this functionality are shown in [Example %s](#get-svdv-data02), below:


```{code} python
:label: get-svdv-data02
:caption: Examples of reading select SV and DV data
:linenos: 

# get some SV data - let's say inflows to Shasta and Oroville
# -- using simple filter terms
cs3_study.SVdata.getSVts(filter=['/I_SHSTA/', '/I_OROVL/']) 

# We can also filter DSS pathnames using regular expressions
# the `append_dataframe = True` option will add the closure term
# data to the new dataframe rather than create a new dataframe (the default)
cs3_study.SVdata.getSVts(filter=["CT_\S*_SV"], filt_method='regex', 
                        append_dataframe=True) 


# we get DV data the same way, just using `DVdata.getDVts`
dv_vars =['/S_SHSTA/','/DEL_CVP_PAG_S/','/DEL_SWP_PMI_S/',
          '/NDO/', '/C_SAC041/']
cs3_study.DVdata.getDVts(filter=dv_vars)

```

The `filter` argument accepts a list of comma-separted strings.
Each string will be compared against the path list extracted from the DSS file, and the data for any matching paths will be returned.
Paths that are not matched will return no data - and, importantly, an error will be raised if no matching data are found at all.

The regular expressions option (using keyword argument `filt_method = 'regex'`) will tell the function to filter the path list using the regular expression string provided.

The `append_dataframe` argument controls whether the function creates a new Pandas dataframe for storing any retrieved time series (`append_dataframe = False`, the default option) or appending the retrieved data to any existing timeseries data that had been previously collected (`append_dataframe = True`).

```{hint} Filtering DSS paths
If you're not familiar with DSS variable naming, each time series is identified with a 7-part forward slash-separated 'path', where each part provides some metadata or information associated with that path. 
In CalSim applications, the convention is that the A-part is always 'CALSIM', and the B- and C-parts provide the WRESL variable name and WRESL variable 'kind'. 

If there is any ambiguity in the B-part, consider bracketing the B-part with forward slashes and/or included the C-part as well. 
This will reduce the likelihood of unwanted variables and data being returned (e.g. using **'S_SHSTA'** will return ~35 variables, like *S_SHSTALEVEL1DV*, *S_SHSTA_1*, *S_SHSTA_DELTA*, etc).
Using **'/S_SHSTA/'** will return just the single final Shasta storage variable. 
```

The `getSVts` and `getDVts` functions read data into a Pandas dataframe that is stored in the `SVdata` or `DVdata` objects, respectively. 
You can access this data by calling the dataframe directly:
    * `SVdata.SVtsDF`: SV time series dataframe
    * `DVdata.DVtsDF`: DV time series dataframe

The following code shows an example of what this looks like with our SV data object.

```python
# all the retrieved timeseries data lives in a pandas dataframe, and can
# be accessed through the SVtsDF component of the SVdata object
svDF = cs3_study.SVdata.SVtsDF

svDF.head()
```

The `svDF.head()` command returns a preview of the top five rows of the dataframe, shown in [Figure %s](#fig-ex-svdf).

```{figure} figs/01calsim/example_SVts_dataframe.png
:label:fig-ex-svdf

Example of a Pandas dataframe in which time series data from a CalSim SV file has extracted and stored.
```

We see that the header structure of the DSS data has been retained using a multi-indexed column. 
This is meant to mimic the information available through the HEC-DSSVue software or how data was imported to Excel using the DSS Excel add-in. 
The index is a monthly time series that uses the end-of-month values consistent with CalSim convention and other DSS tools.

The data in this dataframe can now be used to many different purposes - you can write it out to any format supported by Pandas for dataframes (csvs are probably the most common), you can make graphs and figures, or you can modify the data and write out a new file.
*Note* - the ability to write out time series data in **_CalSimPy_** is limited to only the SV data object. 
This is by design - the DV data should only be modified as part of a simulation.
The next section covers how to modify data and write it to a new DSS file.

## Writing SV data to DSS

The function to write SV data to a DSS file essentially works in reverse to the function for reading that data in.
The function takes the form of *{calsimObject}.SVdata.setSVts({path_to_DSS_file})*, where *{path_to_DSS_file}* is a path to a new (or existing - take care, as you can overwrite data!) DSS file where you want your data to live.

The code shown in [Example %s](#write-sv-data03) walks through the steps one might take to modify data and write it to a new DSS file.

```{code} python
:label: write-sv-data03
:caption: Example workflow for modifying SV data and writing to a new DSS file
:linenos: 

# we'll make copies of objects to avoid overwriting things
from copy import deepcopy

# get the SV dataframe (if we haven't already done so)
svDF = cs3_study.SVdata.SVtsDF 

# make a copy of your dataframe so we can make changes and not 
# modify the original 
svDF_mod = svDF.copy(deep=True)

# create a new variable - specify the full header - if you
# have to do this for many variables, you can simplify that part 
# substantially
svDF_mod['CALSIM','I_SHSTA90pct',
        'INFLOW','1MON','L2020A',
        'PER-AVER','TAF'] = svDF_mod.loc[:, idx[:,'I_SHSTA' ]].values

# filter out just the Februaries
feb_idx = svDF_mod.index.month==2

# do the calculation - let's make February values 90% of the original
svDF_mod.loc[feb_idx, idx[:,'I_SHSTA90pct']] = \
            svDF_mod.loc[feb_idx,idx[:,'I_SHSTA']].values*0.90

# now we can write this data back to a DSS file
# set the path to a new file 
new_dss_fp = r'../example_cs3_study/DSS/input/modified_ishsta.dss'

# copy the calsim object - so we don't modify the original data
cs3_study_copy = deepcopy(cs3_study)

# replace the dataframe with our modified version
cs3_study_copy.SVdata.SVtsDF = svDF_mod 

# and write it out to DSSS using `setSVts` command
cs3_study_copy.SVdata.setSVts(new_dss_fp)


```

The code example does several things in sequence:

1. **Line 9**: This copies the SV dataframe so we don't overwrite the original data
2. **Line 14**: This creates a new variable with a consistent header and a new, unique part-B identifier (*'I_SHSTA90pct'*)
3. **Lines 19-23**: These commands modify the data according to our intent of reducing all February values by 10%
4. **Line 27**: This sets a new DSS path (you would set this to a path specific to your own computer)
5. **Line 30**: This copies the CalSim study object so we (again) aren't overwriting anything. This is not strictly necessary, but is helpful if you want to avoid confusing or modifying files.
6. **Line 33**: This command sets the SV time series dataframe in the new (copied) study to our modified data.
7. **Line 36**: This command writes out our new modified dataframe to the DSS file at the location specified on line 27.

You can check the results in the new DSS file to confirm the changes were made as intended using separate software or code.
A zoomed in view of several years is shown in [Figure %s](#fig-ex-writesvdss) to illustrate that this example did indeed work as intended.


```{figure} figs/01calsim/example_writeModSVdss.png
:label:fig-ex-writesvdss

A zoomed-in view of the monthly Shasta inflow time series using HEC-DSSVue for two versions of the data - the original (blue) and modified (red). 
The red line is slighly lower than the blue in each February, confirming the modification to the DSS time series was performed as intended using **_CalSimPy_**.
```