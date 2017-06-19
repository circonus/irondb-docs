# Changelog

## Changes in 0.8.29
2017-06-19

 * Better internal defaults for raw metrics database, to reduce compaction stalls, improving throughput.
 * Cache SHA256 hashes in topology-handling code to reduce CPU consumption.
 * Fix memory-usage errors in LRU cache for Graphite queries.
 * Fix memory leaks relating to replication journals.
 * Fix for failed deletes due to filename-too-long errors.

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

 * Improve replicate_journal message handling.
 * Speed up journal processing.
 * Increase write buffer and block size in raw database to reduce write stalls.

## Changes in 0.8.13
2017-04-14

 * Reduce CPU usage on journal_reader threads.
 * Fix crash during rollup when rewinding the epoch of a data file.
 * Increase default read buffer size for Graphite listener.
 * Use proper libcurl error defines in replication code.

## Changes in 0.8.12
2017-04-12

 * Remove problematic usage of alloca().
 * Add lz4f support to reconstitute.
