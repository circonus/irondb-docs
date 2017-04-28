# Changelog

## Changes in 0.8.18
2014-04-28

 * Fix potential races in replication.
 * Speed up metric querying.

## Changes in 0.8.17
2014-04-27

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
