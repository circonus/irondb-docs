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
* Favor ZFS striped mirrors over other pool layouts. This provides the highest
  performance in IOPS.
* `W` must be >= 2
* `N` must be >= `W`
* `W` should be >= 3 when `N` >= 6
* `W` should be >= 4 when `N` >= 100

## Storage Space

The system stores three types of data: text, numeric (statistical aggregates),
and histograms. Additionally there are two tiers of data storage: near-term and
long-term. Near-term storage is called the [raw
database](/configuration.md#rawdatabase) and stores at full resolution (however
frequently measurements were collected.) Long-term resolution is determined by
the [rollup configuration](/configuration.md#rollups).

The default configuration for the raw database is to collect data into shards
(time buckets) of 1 week, and to retain those shards for 4 weeks before rolling
them up into long-term storage. At 1-minute collection frequency, a single
numeric stream would require approximately 118 KiB per 1-week shard, or 472 KiB
total, before being rolled up to long-term storage.

These numbers represent uncompressed data. With our default LZ4 compression
setting in ZFS, we see 3.5x-4x compression ratios for numeric data.

The following modeling is based on an observed distribution of all data types,
in long-term storage, across many clients and may be adjusted from time to
time. This would be in addition to the raw database storage above.

| Minimum Resolution | Storage Space / Day | Storage Space / Year |
|:-------------------|--------------------:|---------------------:|
| 10 seconds | 120,000 bytes | 43,020,000 bytes |
| 1 minute   |  20,000 bytes |  7,170,000 bytes |
| 5 minute   |   3,800 bytes |  1,386,000 bytes |

All sizing above represents uncompressed data.

### Sizing Example

Suppose we want to store 100,000 metric streams at 1-minute resolution for 5
years.  We'd like to build a 4-node cluster with a `W` value of 2.

```
T=100,000
N=4
W=2

T * 7,170,000 (bytes/year/stream) * 5 years = 3,585,000,000,000 bytes

3,585,000,000,000 bytes / (1024^3) = 3338 GiB

T * 483,840 (bytes/4 weeks raw/stream) / (1024^3) = 45 GiB

( (3338+45) * W) / N = 1692 GiB per node

1692 GiB / 70% utilization = 2417 GiB of usable space per node

2417 GiB * 2 = 4834 GiB of raw attached storage in ZFS mirrors per node
```

## Hardware Choices

Circonus recommends server-class hardware for all production deployments. This
includes, but is not limited to, features like ECC memory and hot-swappable
hard drives.

 * See [OpenZFS guidelines](http://open-zfs.org/wiki/Hardware) for general
   advice.
   * Specifically, hardware RAID should be
     [avoided](http://open-zfs.org/wiki/Hardware#Hardware_RAID_controllers). ZFS
     should be given access to raw hard drive devices whenever possible.

In addition to the overall storage space requirements above, consideration must
be given to the IOPS requirements. The minimum IOPS required is the primary
write load of ingesting metric data (approximately 12 bytes per measurement
point), but there is additional internal work such as parsing and various
database accounting operations that can induce disk reads beyond the pure
writing of measurement data. After initial ingestion there are other
operations, such as searching, rollups, and maintenance activity like
reconstitution and ZFS scrubbing that require additional IOPS.  Ensure that the
hardware you choose for your nodes has the capacity to allow for these
operations without significantly impacting ongoing ingestion.

ZFS's
[ARC](http://open-zfs.org/wiki/Performance_tuning#Adaptive_Replacement_Cache)
helps by absorbing some portion of the read load, so the more RAM available to
the system, the better.

### Hardware Profiles

The following are sample profiles to guide you in selecting the right
combination of hardware and cluster topology for your needs.

Assumptions:
 * 10-second collection frequency
 * 4 weeks of near-term (full-resolution) storage
 * 2 years of historical data at 1-minute resolution
 * striped-mirror ZFS pool layout

|Streams per 10sec|Write Copies|Total Streams|Node Count|Streams per Node|Physical CPU cores|RAM (GB)|7200rpm spindles|
|----------------:|:----------:|------------:|:--------:|:--------------:|:----------------:|:------:|---------------:|
|   1MM | 3 |   3MM |  5 | 600K | 12 | 128 |  6x 2T |
|  10MM | 3 |  30MM | 15 |  2MM | 24 | 256 | 24x 4T |
| 100MM | 3 | 300MM | 75 |  4MM | 36 | 384 | 45x 4T |
