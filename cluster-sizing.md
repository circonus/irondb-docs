# Calculating Cluster Dimensions

This is intended as a general guide to determining how many nodes and how much
storage space per node you require for your workload. Please [contact
Circonus](/contact.md) if you have questions arising from your specific needs.

## Key Terminology

* `T` is the number of unique metric streams.
* `N` is the number of nodes participating in the cluster.
* `W` is the number of times a given measurement is stored across the cluster.
  * For example, if you have 1 GB of metric data, you must have `W` GB of
    storage space across the cluster.

The value of `W` determines the number of nodes that can be unavailable before
a metric data become inaccessible. A cluster with `W` write copies can survive
`W-1` node failures before data become inaccessible.

Metric streams are distributed approximately evenly across the nodes in the
cluster. In other words, each node is responsible for storing approximately
`(T*W)/N` metric streams. For example, a cluster of 6 nodes with 100K streams
and `W=3` would store about 50K streams per node.

## Rules of Thumb

* Nodes should be operated at no more than 70% capacity.
* Favor ZFS striped mirrors over other pool layouts
* `W` must be >= 2
* `N` must be >= `W`
* `W` must be >= 3 when `N` >= 6
* `W` must be >= 4 when `N` >= 32

## Storage Space

The system stores three types of data: text, numeric (statistical aggregates),
and histograms. The following modeling is based on an observed distribution of
those data elements across many clients and may be adjusted from time to time.

| Minimum Granularity | Storage Space / Day | Storage Space / Year |
|:--------------------|:--------------------|:---------------------|
| 1 minute | 20,000 bytes | 7,170,000 bytes |
| 5 minute | 3,800 bytes | 1,386,000 bytes |

## Example

Suppose we want to store 100,000 metric streams at 1-minute granularity for 5
years.  We'd like to build a 4-node cluster with a `W` value of 2.

```
T=100000
N=4
W=2

T * 7170000 (bytes/year/stream) * 5 years = 3585000000000 bytes

3585000000000 bytes / (1024*1024*1024) = 3338 GiB

(3338 * W) / N = 1669 GiB per node

1669 GiB / 70% utilization = 2384 GiB of usable space per node

2384 GiB * 2 = 4768 GiB of raw attached storage in ZFS mirrors per node
```
