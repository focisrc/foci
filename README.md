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
