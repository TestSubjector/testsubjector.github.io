---
layout: post
title: "A Primer On HDF5 File-Reading In Fortran 90"
date: 2020-09-30
mathjax: true
---
----------------

Recently, I had to learn how to read HDF5 format files in Fortran and implement the same in one of my research group's projects.

The experience was not what I'll call stellar, I was unhappy for much of the duration of the process as I wasn't enjoying the work.
This was primarily because of the opaqueness of the HDF5 official documentation [^1] I was working with.  

However, I was lucky to stumble onto the DOAS group[^2], who had created a package(module) which simplified HDF5 file reading. 
The package itself was too cumbersome for my needs (they had sacrificed performance for accessibility). However the source code they provided gave me the clarity and insight I needed to understand the working of HDF5 file reading.

Below, I've just put into examples, some of the concepts I've learned. Note that this covers the basics (hence the *Primer* in the title of this post) and is aimed at assisting in getting started with HDF5.

----------------
#### Prerequisites
----------------
* FORTRAN 90
* HDF5 Module for FORTRAN
* A PGI/GCC/iFORT Compiler
* An HDF5 file and knowledge of how information is stored in it

----------------
#### HDF5 File Structure
----------------
<script src="https://gist.github.com/TestSubjector/d2ff68b68bf38fb2fdc012dff535fffb.js"></script>

----------------
#### Fortran HDF5 File Reading
----------------

Now lets come to the main part. We will use the sample file structure given in the `HDF5.js` example above as the targeted file. 
Additionally, I'll also be listing the variables required by the subroutines(functions) in their respective code blocks.
In actual Fortran code, all of those variables mentioned have to be declared before any subroutine call can occur.  
   
First, before anything else we need to initialize the HDF5 library routines -  

```fortran
INTEGER :: ErrorFlag ! Output variable

CALL h5open_f(ErrorFlag)

IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error initialising HDF routines"
    return
ENDIF
```

Now, we need to open the targeted HDF5 file -
  
```fortran
Character(len=65) :: part_grid ! Input variable
INTEGER(HID_T)    :: file_id   ! Output variable

part_grid = 'point.h5' ! Refer to line 5 of HDF5.js 
! which gives the filename
    
CALL h5fopen_f(part_grid, H5F_ACC_RDONLY_F, file_id, ErrorFlag)

IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error opening HDF file"
    return
ENDIF
```

Then, we need to open the root GROUP "/" present in line 6 of `HDF5.js`, which encloses all other data - 
  
```fortran
CHARACTER(LEN = 1)  :: rootname ! Input variable
INTEGER(HID_T)      :: root_id  ! Output variable

rootname = "/"

CALL h5gopen_f(file_id, rootname, root_id, ErrorFlag)

IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error opening root group"
    return
ENDIF
```
  
Here, `file_id` is the same variable identifier initialized by the `h5open_f()` subroutine. This gives the HDF5 library the link between the file and requested group to be opened. With the preliminaries over with, let's try to read some data.  

Suppose we want to read the data in ATTRIBUTE "total". Looking at the `HDF5Hierarchy.md`, we can see that the specific ATTRIBUTE is enclosed by GROUP "1" which is itself enclosed by the GROUP "/" we just opened.  

Therefore, we need to open GROUP "1" using the variable identifier `root_id` we received from opening the root GROUP "/" -
  
```fortran
character(len=10) :: main_group ! Input variable
INTEGER(HID_T)    :: group_id   ! Output variable

main_group = '/'//'1'

CALL h5gopen_f(root_id, main_group, group_id, ErrorFlag)

IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error opening Group 1"
    return
ENDIF
```

Then use that newly initialized identifer `group_id` to open ATTRIBUTE "total" -

```fortran
character(len=10) :: total_attribute ! Input variable
INTEGER(HID_T)    :: a_id            ! Output variable
    
total_attribute = 'total'

CALL h5aopen_name_f(group_id, total_attribute, a_id, ErrorFlag)
    
IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error opening total attribute "
    return
ENDIF
```

Finally, now that we have access to the ATTRIBUTE we want, we need to extract the data from it. 
Looking at the contents, we see that the datatype is of `H5T_STD_I32LE`, which is essentially an integer[^3].
Similarly, it stores only one element. With this information we can read the information from the attribute.

```fortran
INTEGER(hsize_t), DIMENSION(1) :: dims          
INTEGER                        :: total_points  

dims=1                                          
    
CALL h5aread_f(a_id, H5T_NATIVE_INTEGER, total_points, dims,& 
& ErrorFlag)  

! H5T_NATIVE_INTEGER is a part of some mental gymnastics 
! related to making sure your integers 
! are the same as the file's integers. It's a datatypes/compiler 
! thing and more info can be found for the curious in the 
! References section at the end of this blog, under the 
! HDF5 Predefined Datatypes.
```

The required data to be read, which is given at line 12 in `HDF5.js` is `9600`. This will now be stored in the `total_points` variable.

Finally, once everything is read, it's in good spirits to close all the opened structures.

```fortran
CALL h5aclose_f(a_id, ErrorFlag)
CALL h5gclose_f(group_id,ErrorFlag)
CALL h5gclose_f(root_id,ErrorFlag)
CALL h5fclose_f(file_id,ErrorFlag)
CALL h5close_f(ErrorFlag)

IF (ErrorFlag.lt.0) THEN
    ErrorMessage=" *** Error closing HDF routines"
    return
ENDIF
```

This concept can then be expanded to read datasets, multiple dimensional chunks of data. Then there's helper subroutines like `h5aget_space_f()` to determine dataspace (data stored in an ATTRIBUTE or DATASET) and dataspace dimensionality, at the time of execution rather than the approach of setting `dims=1` which I have used here.

However, this is just supposed to be a small beginner friendly introduction to the topic. Maybe sometime later I'll add a Part 2 to this for more advanced operations, but no plans as of now. 

For reference, you can find more extensive usage of the HDF5 code here [^4].  

----------------
**References:**

[^1]: [HDF5 Official Documentation](https://portal.hdfgroup.org/display/HDF5/HDF5)
[^2]: [The DOAS Group HDF5 module](http://uv-vis.aeronomie.be/software/tools/hdf5read.php)
[^3]: [HDF5 Predefined Datatypes](https://support.hdfgroup.org/HDF5/doc/RM/PredefDTypes.html)
[^4]: [Github Repository with HDF5 usage](https://github.com/Nischay-Pro/mfcfd/blob/hdf5-sec-order/src_mpi_serial/point_preprocessor.F90#L177)
