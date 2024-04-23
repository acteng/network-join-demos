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

``` r
if (!file.exists("open_roads_example.zip")) {
    message("You lack open roads data locally")
    u = "https://api.os.uk/downloads/v1/products/OpenRoads/downloads?area=GB&format=GeoPackage&redirect"
    f = "oproad_gpkg_gb.zip"
    if (!file.exists(f)) download.file(u, f)
}
```

    You lack open roads data locally

# Basical spatial join
