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


## Data Structure and Programming Paradigms

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
For example, if we want to unfold the polarization components *I*,
*Q*, *U*, and *V*, the above table is *equivalent* to:

Source | Baseline  | *I*              | *Q* | *U* | *V*
------ | --------- | ---------------- | --- | --- | ---
M87    | ALMA-ALMA | `Vis[s,t,b,c,d]` | ... | ... | ...
M87    | ALMA-APEX | `Vis[s,t,b,c,d]` | ... | ... | ...
...    | ...       | ...              | ... | ... | ...

This unfolding does not require a significant data movement.
