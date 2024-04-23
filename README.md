# Testing network joining functions
Robin Lovelace

# Introduction

Joining data is key to data science, allowing value to be added to
disparate datasets by combining them.

There are various types of join, including based on shared ‘key’ values
and shared space for spatial joins. However, neither of these join types
works for joining network data of the type shown below, which represents
2 separate networks with different but related geometries (source: [ATIP
browse
tool](https://acteng.github.io/atip/browse.html?style=streets#13.32/53.79562/-1.6874)).

![](images/paste-1.png)

Imagine you want to know what kind of cycle infrastructure is associated
with each segment of the MRN. That’s the kind of problem that network
joins can tackle.

This guide outlines the challenges of network joining and demonstrates
implementation-agnostic solutions.

It’s based on previous work:

- The networkmerge project (and related
  [parenx](https://github.com/anisotropi4/parenx) Python package
  available on pip): <https://nptscot.github.io/networkmerge/>

- The rnetmatch approach, which has been implemented in Rust with a
  nascent R wrapper (there are plans for a Python wrapper)

- An approach in JavaScript at
  <https://github.com/acteng/amat/tree/main/pct_lcwip_join>, described
  at <https://github.com/acteng/amat/blob/main/js/model.md#pct-join>

We’ll use data from the Propensity to Cycle Tool and the
[OpenRoads](https://osdatahub.os.uk/downloads/open/OpenRoads) dataset as
an example.

# Example datasets

Datasets were take from a few case study areas.

## Thornbury, West Yorkshire

    Reading layer `open_roads_thornbury' from data source 
      `/home/robin/github/acteng/network-join-demos/data/open_roads_thornbury.gpkg' 
      using driver `GPKG'
    Simple feature collection with 421 features and 20 fields
    Geometry type: LINESTRING
    Dimension:     XY
    Bounding box:  xmin: 418537.8 ymin: 433158 xmax: 420517 ymax: 435028.6
    Projected CRS: OSGB36 / British National Grid

    Reading layer `pct_thornbury' from data source 
      `/home/robin/github/acteng/network-join-demos/data/pct_thornbury.gpkg' 
      using driver `GPKG'
    Simple feature collection with 95 features and 1 field
    Geometry type: LINESTRING
    Dimension:     XY
    Bounding box:  xmin: 418585.9 ymin: 433201.9 xmax: 420499.9 ymax: 434468.5
    Projected CRS: OSGB36 / British National Grid

![](README_files/figure-commonmark/load-data-thornbury-1.png)

A first step, to speed-up the join and reduce the size of the data, can
be to keep only the records in the target ‘x’ dataset that are relevant.
After this filtering step, the datasets look like this:

![](README_files/figure-commonmark/subset-data-1.png)

Of the four options, the third (with a distance of 20) looks like the
best compromise between omitting unwanted links while retaining the
majority of the network.

The most appropriate distance depends on your data and use case, it may
be worth keeping more of the ‘x’ network than you need and using the
join to filter out unwanted links (the subsetting stage is not
essential).

# Basic spatial join

A simple approach to joining the two networks is with a simple spatial
join, using one of the available ‘binary predicates’, such as
`st_intersects`, `st_within` (relevant for buffers), `st_contains` or
`st_touches`.

The results when the `flow` attributes are summed are shown below:

![](README_files/figure-commonmark/spatial-join-1.png)

Unfortunately the results are way out: the total flow in the joined
network using this basic join approach is less than half (48 %) the
total flow in the original network.
