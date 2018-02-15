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
metric data become inaccessible. A cluster with `W` write copies can survive
`W-1` node failures before a partial data outage will occur.

Metric streams are distributed approximately evenly across the nodes in the
cluster. In other words, each node is responsible for storing approximately
`(T*W)/N` metric streams. For example, a cluster of 4 nodes with 100K streams
and `W=2` would store about 50K streams per node.

## Rules of Thumb

* Nodes should be operated at no more than 70% capacity.
* Favor ZFS striped mirrors over other pool layouts
* `W` must be >= 2
* `N` must be >= `W`
* `W` should be >= 3 when `N` >= 6
* `W` should be >= 4 when `N` >= 100

## Storage Space

The system stores three types of data: text, numeric (statistical aggregates),
and histograms. Additionally there are two tiers of data storage: near-term and
long-term. Near-term storage is called the [raw
database](/configuration.md#rawdatabase) and stores at full resolution (however
frequently measurements were ingested.) Long-term resolution is determined by
the [rollup configuration](/configuration.md#rollups).

The default configuration for the raw database is to collect data into shards
of 1 week, and to retain those shards for 4 weeks before rolling them up into
long-term storage. At 1-minute resolution, a single numeric stream would
require approximately 118 KiB per 1-week shard, or 472 KiB total, before being
rolled up to long-term storage.

The following modeling is based on an observed distribution of all data types,
in long-term storage, across many clients and may be adjusted from time to
time. This would be in addition to the raw database storage above.

| Minimum Resolution | Storage Space / Day | Storage Space / Year |
|:-------------------|:--------------------|:---------------------|
| 1 minute | 20,000 bytes | 7,170,000 bytes |
| 5 minute | 3,800 bytes | 1,386,000 bytes |

These numbers represent uncompressed data. With our default LZ4 compression
setting in ZFS, we see 3.5x-4x compression ratios for numeric data.

## Example

Suppose we want to store 100,000 metric streams at 1-minute resolution for 5
years.  We'd like to build a 4-node cluster with a `W` value of 2.

```
T=100000
N=4
W=2

T * 7170000 (bytes/year/stream) * 5 years = 3585000000000 bytes

3585000000000 bytes / (1024^3) = 3338 GiB

T * 483840 (bytes/4 weeks raw/stream) / (1024^3) = 45 GiB

( (3338+45) * W) / N = 1692 GiB per node

1692 GiB / 70% utilization = 2417 GiB of usable space per node

2417 GiB * 2 = 4834 GiB of raw attached storage in ZFS mirrors per node
```
