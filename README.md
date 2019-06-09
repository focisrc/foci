# Foci

`Foci` is a toolkit for Fast Operations in Computational
Interferometry.
It is designed as a "middle-ware" that can be used by other high-level
python packages.


## Data Processing Steps, Data Volumes, and Needs

Data processing steps in computational interferometry can naturally be
categorized based on data volumes.
Taking the [Event Horizon Telescope (EHT)](
http://eventhorizontelescope.org), a global [Very-long-baseline
interferometry (VLBI)](
https://en.wikipedia.org/wiki/Very-long-baseline_interferometry)
experiment, as an example, the data volumes scale from petabytes to
kilobytes along its data processing pipeline.

- At the telescope stations, radio waves from accretion flows around
  black holes are recorded on hard-disks, summing up to petabytes (PB)
  of data for an observation campaign.
- The hard-disks are then shipped to special supercomputers called
  correlators, where the radio wave signals are correlated and
  averaged, reducing the the data volume to terabytes (TB).
- These correlated visibilities are then transferred to the Cloud,
  fringe fitted using multiple calibration pipelines, and further
  reduced to gigabytes (GB) scale.
- A-priori and polarimetry calibrations, and further frequency and
  time averaging, are then applied to the fringe fitted data,
  resulting a condensed megabyte (MB) scale data set.
- Finally, this MB scale data set is used in image reconstruction and
  model comparison, results kilobytes (kB) of images or plots, and
  helps astronomers answer the project's science questions.

Clearly, with data volumes spread such a large range, very different
computation approaches are needed in different steps.

- While edge computing may help reducing the data recorded at
  telescope stations, the correlation step will likely require
  distributed computing on computer clusters---either in the form of
  more traditional high-performance computing or the new paradigm of
  cloud computing.
- Thanks to cloud computing, computer nodes are become fatter and it's
  common to have 100 cores on a single server.
  With the help of accelerators such as GPUs and FPGAs, fringe-fitting
  and calibrations will likely require parallel computer within a
  single powerful server.
- The MB scale frequency and time averaged data set, while small in
  size, is directly connected to the science.
  Hence, interactive data analysis with good data visualization is
  necessary.


## Visibility Data

In order to satisfy the different data processing needs, `foci`
seamless integrates different technologies and programming frameworks
into a consistent interface of visibility data.

To provide a high level abstraction of the correlated visibility data,
it is natural to separate the "sparse", point-like parameters:

* Source
* Baseline

the "dense discrete" labels:

* Polarization component
* Scan
* Band

and the "dense continue" (but discretized) variables:

* Time (within a scan)
* Channel frequency
* Subchannel frequency

For sparse parameters, it is clear that we should use particle-based
data structure.
I.e., define fields/columns in a structure/table and populate the
list/rows with valid values.
Conversely, for the dense continue variables, arrays are ideal.
However, how to handle the dense discrete labels are less obvious.
We need to consider them case-by-case and provide a well-defined
transformation between the two different representations.

Let `p`, `s`, `t`, `b`, `c`, and d be the indices for polarization
component, scan, time, band, channel, and subchannel.
If we represent `p`, `s`, and `b` as array dimension, a VLBI
visibility data set can be represented by:

Source | Baseline  | Visibility Data Array
------ | --------- | ---------------------
M87    | ALMA-ALMA | `Vis[p,s,t,b,c,d]`
M87    | ALMA-APEX | `Vis[p,s,t,b,c,d]`
...    | ...       | ...

There are many ways to implement the above data table in memory,
including embedding the arrays, use pointers to memory, or store all
visibility arrays in a global array access it by the row number of the
table.
Anyway, to "unfold" one of these dense discrete labels, we turn the
array dimension into multiple columns in the data table.
For example, if we want to unfold the scan dimension, the above table
is *equivalent* to:

Source | Baseline  | s0               | s1  | s2  | ...
------ | --------- | ---------------- | --- | --- | ---
M87    | ALMA-ALMA | `Vis[p,t,b,c,d]` | ... | ... | ...
M87    | ALMA-APEX | `Vis[p,t,b,c,d]` | ... | ... | ...
...    | ...       | ...              | ... | ... | ...

This unfolding can be made so that it does not require significant
data movement.
The operation that requires data movement is to turn the columns into
rows:

Source | Baseline  | Scan | Visibility Data Array
------ | --------- | ---- | ---------------------
M87    | ALMA-ALMA | t0   | `Vis[p,t,b,c,d]`
M87    | ALMA-ALMA | t1   | `Vis[p,t,b,c,d]`
M87    | ALMA-ALMA | ...  | `Vis[p,t,b,c,d]`
M87    | ALMA-APEX | t0   | `Vis[p,t,b,c,d]`
...    | ...       | ...  | ...

This is often referred as the "unpivot" or "melt" operation.
Although it is possible to unpivot any dimension in the visibility
data array, the dense discrete labels are the more natural dimensions
to unpivoate.


## Data Structure and Programming Paradigms

Base on the above discussion, we use `xarray.DataArray` for visibility
array; we also subclass `xarray.DataSet` to `VisData` for our overall
data interface.

The default dimensions for visibility array are polarization
component, time, band, channel, and subchannel.
`Foci` provides methods to split and merge visibility arrays along
different dimensions for a uniform interface for averaging.

The default columns for `VisData` are Source, Baseline, Scan ID/Time,
and visibility array, as listed in the above table.
The "metadata" in the first 4 columns are enough for performing most
of the query operations and allow us to derive the *uv* coordinates.
`Foci` also provides methods to melt, pivot, and manipulate `VisData`
for more complicated operations.
