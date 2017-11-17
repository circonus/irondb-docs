# Changelog

## Changes in 0.10.13
2017-11-17

 * Fix potential double free crash upon query cache expiry

## Changes in 0.10.12
2017-11-15

 * Lock free cache for topology hashes
 * Fix graphite response when we have no data for a known metric name

## Changes in 0.10.11
2017-11-13

 * Disable cache for topology hashes due to live lock

## Changes in 0.10.10
2017-11-13

 * Validate incoming /metrics/find queries are well formed
 * Move query cache to an LFU

## Changes in 0.10.9
2017-11-10

 * Fix for crash on extremely long /metrics/find queries

## Changes in 0.10.8
2017-11-09

 * IRONdb now supports listening via the [Pickle protocol](configuration.md#pickle-listener).

Multiple `whisper2nnt` changes:
 * Add `--writecount` argument for limiting the number of data points submitted per request
 * Submit to the primary owning node for a given metric
 * Disable HTTP keepalive
 * Add `--find_closest_name` parameter. This is needed for sites that do name manipulation via modules in the metric\_name\_db and submit metrics with one name but search on them with another name. For example, a metric would get submitted that resembles `foo.bar_avg` and returned from the metric\_name\_db as `foo.bar`.  Ingestion of whisper data has to use the `foo.bar_avg` name but whisper files on disk do not follow this format.  To combat this a new switch which uses the
`/graphite/metrics/find` url to lookup an already ingested name based on the whisper name as a prefix and uses that name for metric submission under NNT.

## Changes in 0.10.7
2017-11-03

 * Prevent OOM conditions when there are large chunks of new metric\_name\_db values
 * Pre-populate the metric\_name\_db cache on startup
 * Replace usage of fnmatch with PCRE, fixing some cases where fnmatch fails
 * Allow proxied metrics/find queries to utilize the cache

## Changes in 0.10.6
2017-10-31

 * Increased parallelism in metric\_name\_db maintenance
 * whisper2nnt: include in submission those archives with a period coarser than the minimum
 * whisper2nnt: re-raise exception after two consecutive submission failures
 * Better error handling for topology loading failures
 * Several memory-related bug fixes

Documentation changes:
 * The IRONdb Relay installer no longer insists on ZFS, and creates directories instead.
 * Explicitly document that cluster resize/rebalance does not support changes to "sidedness". A new cluster and full reconstitute is required for changing to/from a sided cluster.

## Changes in 0.10.5
2017-10-24

 * Eliminate lock contention on a hot path when debugging is not enabled.
 * Correct a logic error in choosing the most up-to-date node when proxying.
 * Fix escaped wildcard queries when proxy-querying leaf nodes.
 * Log-and-skip rather than crash on flatbuffer read errors.
 * Crash fix for stack underflow.
 * Several whisper2nnt fixes:
   * Retry submissions when a connection to IRONdb is reset.
   * Sort output before submitting to IRONdb, avoids rewinding epoch on numeric data files.
   * New arguments to help with debugging: `--debug`, `--noop`
 * Includes [libmtev fix](https://github.com/circonus-labs/libmtev/pull/328) for a startup issue with file permissions.

## Changes in 0.10.4
2017-10-12

 * Fixes for reconstitute status handling.
 * Fix use-after-free in graphite GET path.

Documentation changes:
 * Add documentation for [irondb-relay](irondb-relay.md), a cluster-aware carbon-relay/carbon-c-relay replacement.
 * Merge content for deleting numeric metrics and entire checks.

## Changes in 0.10.3
2017-10-06

 * Ensure metrics injected via whisper2nnt tool are visible.

## Changes in 0.10.2
2017-10-05

 * Another late-breaking fix to speed up writes to the metric\_name\_db.

## Changes in 0.10.1
2017-10-05

 * Late-breaking optimization to avoid sending /metrics/find requests to down nodes.

## Changes in 0.10.0
2017-10-04

 * New replication protocol format, utilizing Google FlatBuffers. **This is a backward-incompatible change.** A typical rolling upgrade should be performed, but nodes will not send replication data until they detect FlatBuffer support on the other end. As a result, there may be increased replication latency until all nodes are upgraded.
 * Improved error handling during reconstitute.

Documentation changes:
 * New page documenting [cluster resizing](resizing-clusters.md) procedures.
 * Add system tuning suggestions to the [Installation page](installation.md#system-tuning).

## Changes in 0.9.11
2017-09-22

 * Reconstitute fixes.
 * Fix a bug that prevents a graphite listener from running properly with SSL/TLS on.

## Changes in 0.9.10
2017-09-15

 * Fix bugs in proxying graphite requests where unnecessary work was being triggered.
 * Generated JSON was badly formatted when mixing remote and local results.
 * Add internal timeout support for graphite fetches.
 * Optimize JSON construction for proxy requests.
 * Enable gzip compression on reconstitute requests.

Documentation changes:
 * New page documenting the [configuration files](configuration.md).

## Changes in 0.9.9
2017-09-13

 * Split graphite metric fetches into separate threads for node-local vs. remote to improve read latency
 * Provide a configuration option for toggling LZ4 compression on journal sends (WAL replay to other cluster nodes). The default is on (use compression) and is best for most users.
   * To disable compression on journal sends, set an attribute `send_compressed="false"` on the `<journal>` node in `irondb.conf`.

Documentation changes:
 * Added instructions for [rebuilding failed or damaged nodes](rebuilding-nodes.md)

## Changes in 0.9.8
2017-09-11

 * Optimize JSON processing on metrics\_find responses.
 * Additional fixes to timeouts to prevent cascading congestion on metrics\_find queries.

## Changes in 0.9.7
2017-09-08

 * Fix for potential thundering herd on metrics\_find queries

## Changes in 0.9.6
2017-09-07

 * Fix a performance regression from 0.9.5 in topology placement calculations
 * Various minor fixes

## Changes in 0.9.5
2017-09-05

 * Fix lookup key for topology in flatbuffer-based ingestion. Flatbuffer ingestion format is currently only used by the experimental irondb-relay.
 * Update to new libmtev config API

## Changes in 0.9.4
2017-08-18

 * Various fixes

## Changes in 0.9.3
2017-08-16

 * Fix race condition on Linux with dlopen() of libzfs
 * Crash fix: skip blank metric names during rollup
 * Return the first level of metrics\_db properly on certain wildcard queries
 * More efficient Graphite metric parsing

## Changes in 0.9.2
2017-08-04

 * Improve query read speed when synthesizing rollups from raw data
 * Fix double-free crash in handling of series\_multi requests

## Changes in 0.9.1
2017-08-01

 * Fix crash in topology handling for clusters of more than 10 nodes
 * Check topology configuration more carefully on initial import
 * Various stability fixes
 * Document network ports and protocols required for operation

## Changes in 0.9.0
2017-07-13

 * Support for parallelizing rollups, which can be activated by adding a
   "rollup" element to the `<pools>` section of `irondb.conf`, with a
   "concurrency" attribute:
    ```
    <pools>
      ...
      <rollup concurrency="N"/>
    </pools>
    ```
   where `N` is an integer in the range from 1 up to the value of `nnt_put`
   concurrency but not greater than 16. If not specified, rollups will remain
   serialized (concurrency of 1). A value of 4 has been shown to provide the
   most improvement over serialized rollups. 
 * Fix for watchdog-panic when fetching large volumes of data via graphite endpoints.
 * Stop stripping NULLs from beginning and end of graphite responses.
 * Do not return graphite metric data from before the start of collection for that metric.
 * Optimization for graphite fetches through the storage finder plugin.
 * Changes to support data ingestion from new irondb-relay.

## Changes in 0.8.35
2017-06-27

 * Add an option to not use database rollup logic when responding to graphite queries

## Changes in 0.8.34
2017-06-26

 * Throughput optimizations

## Changes in 0.8.33
2017-06-26

 * Fix a bug in database comparator introduced in 0.8.30

## Changes in 0.8.32
2017-06-22

 * Fix a bug with ZFS on Linux integration in the admin UI that caused a segfault on startup.

## Changes in 0.8.31
unreleased

## Changes in 0.8.30
2017-06-21

 * Optimizations for raw data ingestion.
 * Better internal defaults for raw metrics database, to reduce compaction stalls, improving throughput.
 * Cache SHA256 hashes in topology-handling code to reduce CPU consumption.
 * Fix memory-usage errors in LRU cache for Graphite queries.
 * Fix memory leaks relating to replication journals.
 * Fix for failed deletes due to filename-too-long errors.

## Changes in 0.8.29
unreleased

## Changes in 0.8.28
unreleased

## Changes in 0.8.27
2017-06-12

 * Fix a bug that caused contention between reads and writes during rollup.
 * Reduce contention in the raw database write path.

## Changes in 0.8.26
2017-06-02

 * Fix LRU-cache bug for metric queries.

## Changes in 0.8.25
2017-05-31

 * Graphite request proxying preserves original start/end timestamps.
 * Increase replication performance by bulk-reading from the write-ahead log.
 * Improve reconstitute performance.
 * Fix several memory leaks.
 * Note: 0.8.24 was an unreleased internal version. Its changes are included here.

## Changes in 0.8.23
2017-05-18

 * Cache /metrics/find queries.
 * Improved journaling performance.
 * Additional bug fixes.

## Changes in 0.8.22
2017-05-16

 * Efficiency improvement in Graphite queries; we now strip NULLs from both ends of the returned response.
 * Fix a bug in Graphite query that would return a closely related metric instead of the requested one.
 * Fix a bug that caused us to request millisecond resolution when zoomed out too far, and 1-day would be better.
 * First draft of a progress UI for reconstitute.

## Changes in 0.8.21
2017-05-15

 * Inspect and repair write-ahead journal on open.
 * Add a statistic for `total_put_tuples`, covering all metric types.
 * (libmtev) Use locks to protect against cross-thread releases.

## Changes in 0.8.20
2017-05-10

 * Fix for brace expansion in Graphite metric name queries.
 * Resume in-progress rollups after application restart.
 * Improved reconstitute handling.
 * Minor UI fix for displaying sub-minute rollups.
 * Crash and memory leak fixes.

## Changes in 0.8.19
2017-05-03

 * Lower default batch size for replication log processing from 500K to 50K messages. Can still be tuned higher if necessary.
 * Improve ingestion performance in the Graphite listener.

## Changes in 0.8.18
2017-04-28

 * Fix potential races in replication.
 * Speed up metric querying.

## Changes in 0.8.17
2017-04-27

 * (libmtev) Crash fix in HTTP request handling.
 * Disable watchdog timer during long-running operations at startup.
 * Limit writing metrics forward into new time shards.
 * Add multi-threaded replication.

## Changes in 0.8.16
2017-04-24

 * Support brace expansion and escaped queries for Graphite requests.
 * Faster reconstituting of raw data.
 * Fix metric name handling during reconstitute.

## Changes in 0.8.15
2017-04-20

 * Move Graphite listener connection processing off the main thread to avoid blocking.

## Changes in 0.8.14
2017-04-19

 * Improve replicate\_journal message handling.
 * Speed up journal processing.
 * Increase write buffer and block size in raw database to reduce write stalls.

## Changes in 0.8.13
2017-04-14

 * Reduce CPU usage on journal\_reader threads.
 * Fix crash during rollup when rewinding the epoch of a data file.
 * Increase default read buffer size for Graphite listener.
 * Use proper libcurl error defines in replication code.

## Changes in 0.8.12
2017-04-12

 * Remove problematic usage of alloca().
 * Add lz4f support to reconstitute.
