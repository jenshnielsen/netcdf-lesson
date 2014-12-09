# Introducing NetCDF

Recall the data from the very first lesson on Python:

```
0,0,1,3,1,2,4,7,8,3,3,3,10,5,7,4,7,7,12,18,6,13,11,11,7,7,4,6,8,8,4,4,5,7,3,4,2,3,0,0
0,1,2,1,2,1,3,2,2,6,10,11,5,9,4,4,7,16,8,6,18,4,12,5,12,7,11,5,11,3,3,5,4,4,5,5,1,1,0,1
0,1,1,3,3,2,6,2,5,9,5,7,4,5,4,15,5,11,9,10,19,14,12,17,7,12,11,7,4,2,10,5,4,2,2,3,2,2,1,1
0,0,2,0,4,2,2,1,6,7,10,7,9,13,8,8,15,10,10,7,17,4,4,7,6,15,6,4,9,11,3,5,6,3,3,4,2,3,2,1
0,1,1,3,3,1,3,5,2,4,4,7,6,5,3,10,8,10,6,17,9,14,9,7,13,9,12,6,7,7,9,6,3,2,2,4,2,0,1,1
```

This is stored in the comma-separated-value (CSV) format.

> *Question: What are the problems with the CSV format?*
> 
> * *We don't know what the dimensions are (can anyone even remember this from two days ago?),*
> * *We don't what the units are,*
> * *We can only store two dimensional data,*
> * *Not very efficient are storing lots of data.*

NetCDF is a data format which solves all of these problems. With NetCDF:

* Every variable is labelled so that you know what the data represents;
* You can store metadata, like the units that the data is representated in, and how missing values are represented;
* Data can have any number of dimensions. E.g You could store the temperature and moisture content of soil at different depths, across a grid of latitude and longitude, over time;
* It compresses data.

If you share and publish your data in the NetCDF format then it will contain all of the context needed to make sense of the data, even if they don't have any other external information. NetCDF files come with context about the data they represent, and context is everything! Compare this to the CSV file above, which if you found after a year without any other information, you'd have no idea what data it contained.

> ### Challenge
>
> *Think about a data set that you might work with in your research. Write down all of the information that you would need to pass along with that data set to make sure that someone else would understand what the data represented, and how to use it.*
>
> **Get students to shout out a list and write them up somewhere.**

The downside is that NetCDF files are not human readable like CSV files. It's harder to write our own scripts to read and write to NetCDF files directly like we did with the CSV file in the Python lesson. Fortunately we don't have to! There is a comprehensive library for Python, and other languages, that lets us easily work with NetCDF files.

### Lesson Objectives

* Use the NetCDF library in Python to read in and explore data stored in NetCDF files;
* Write data out into NetCDF files for storage;
* Basic manipulation of large data sets, like merging two NetCDF files, or splitting a data set apart into smaller files;
* Plot data that has been read in from NetCDF files.

# Reading NetCDF Data

The first thing to do if we want to read in NetCDF data in Python is to import the netCDF4 module:

```
import netCDF4
```

Recall that we can always ask Python for help with anything, so if you ever need to find out more about using NetCDF with Python, just run:

```
help(netCDF4)
```

The help for netCDF4 comes with a complete tutorial, which you can also view online: http://unidata.github.io/netcdf4-python/.

To read in a NetCDF file, we ask the NetCDF library to open a *Dataset*, which is the term used for a collection of variables, which can all have different units (temperatures, concentrations, etc), and be grouped into combinations of different dimensions (over time, lat & long, depth, etc).

```
dataset = netCDF4.Dataset("HadISST1_SST_update.nc", mode="r")
```

Recall that an advantages of NetCDF is that they give the data context with additional metadata. We can see some of that straight away:

```
print dataset
```

```
<type 'netCDF4.Dataset'>
root group (NETCDF3_CLASSIC data model, file format UNDEFINED):
    Title: Monthly version of HadISST sea surface temperature component
    description: HadISST 1.1 monthly average sea surface temperature
    institution: Met Office Hadley Centre
    ...
```

What we're really interested in though is the raw numbers. Let's see what variables this NetCDF file contains:

```python
print dataset.variables
```
```python
OrderedDict([(u'time', <netCDF4.Variable object at 0x10e35db00>), (u'time_bnds', <netCDF4.Variable object at 0x10e35db98>), (u'latitude', <netCDF4.Variable object at 0x10e35dc30>), (u'longitude', <netCDF4.Variable object at 0x10e35dcc8>), (u'sst', <netCDF4.Variable object at 0x10e35dd60>)])
```
Or more cleanly:
```python
print dataset.variables.keys()
```
```python
[u'time', u'time_bnds', u'latitude', u'longitude', u'sst']
```

A variable in NetCDF is some quantity, like a temperature, or measure of inflammation (as in the Python lesson). A variable doesn't just contain a single value, but instead contains a multi-dimensional array of values with specified dimensions. For examples, a `temperature` variable might be stored as a 2 dimensional array, where one dimension is latitudes, and one dimension is longitudes. The `temperature` variable would hold a value for each combination of latitude and longitude points. If you were measuring moisture in the soil at a single location, you might have a `moisture` variable, where one dimension was soil depth, and another dimension was time.

The `sst` variable in our NetCDF file stores the sea surface temperature. We might not have known that, but fortunately the metadata in the NetCDF file tells us:

```python
print dataset.variables["sst"]
```
```python
<type 'netCDF4.Variable'>
float32 sst(time, latitude, longitude)
    _FillValue: -1e+30
    standard_name: sea_surface_temperature
    long_name: sst
    units: C
    cell_methods: time: lat: lon: mean
    missing_value: -1e+30
unlimited dimensions: time
current shape = (9, 180, 360)
filling off
```

The meta data also tells us other useful information like the units that the temperature is stored in (Celcius), and what the different dimensions in the 3 dimensional array represent (time, latitude, longitude).

Let's see what this data actually looks like. We can plot a 2 dimensional array as an image in Python using the `imgshow` function in matplotlib. Let's plot the sea surface temperature at the first time step in our data:

```python
import matplotlib.pyplot as plt
plt.imshow(dataset.variables["sst"][0,:,:])
```
