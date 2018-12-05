# Changelog

## Changes in 0.14.7
2018-12-05

 * Fix a bug where reconstitute process could get deadlocked and
   not make progress.
 * Fix a potential crash that could occur when reconstituting
   surrogate data.
 * Fix a bug where deleting a metric on a system would not remove
   the surrogate entry if the metric was not local to the node.

## Changes in 0.14.6
2018-12-03

 * Fix bug where text and histogram data transfer could get hung
   during reconstitute.
 * Fix memory ordering related crash in string intern implementation
   (libmtev).

## Changes in 0.14.5
2018-11-30

 * Reclassify an error message as a debug message - message occurs
   in a situation that is not a malfunction and can fill the logs.

## Changes in 0.14.4
2018-11-29

 * Fix crash in metric serialization.

## Changes in 0.14.3
2018-11-29

 * Several memory leaks fixed.
 * Fix reconstitute bug edge case where certain metric names would cause the
   reconstitute to spin/cease progress.
 * Fix bug where certain HTTP requests could hang.
 * Change default raw db conflict resolver to allow overriding old data with
   flatbuffer data from a higher generation.
 * Documentation: Add configuration section describing the surrogate database
   and its options.
 * Documentation: Mark `/read` numeric API as deprecated. The [rollup API](/api/read-rollup.md)
   should be used instead.

## Changes in 0.14.2
2018-11-19

 * Several memory leaks fixed.
 * Improved memory utilization.
 * Performance improvements.
 * Increased speed of surrogate cache loading at startup.
 * Add `snowthsurrogatecontrol` tool, which allows offline
   review and modification of the surrogate database.

## Changes in 0.14.1
2018-11-09

 * Improvements to raw-to-NNTBS rollup speeds.
 * Fix error messages that were printing an uninitialized variable.
 * Handle escaped Graphite expansions that are leaves.
 * Performance improvements via smarter use of locking.
 * More aggressive memory reclamation.

## Changes in 0.14.0
2018-11-01

 * Change some internal HTTP response codes to be more REST compliant/accurate.
 * Improve error checking when opening NNTBS timeshards.
 * Improve surrogate DB startup informational logging.
 * Various memory usage optimizations to reduce the amount of memory needed
   for snowthd to operate.
 * Remove global variables from Backtrace.io traces.
 * Add ability to delete surrogates from the system that are no longer used.
 * Remove temporary files used during reconstitute - there were a handful of
   files staying on disk and taking up space unnecessarily.
 * Increase timeout for pulling raw data during reconstitutes.
 * Move duplicate startup message to debug log - not actually an error, so
   should not be reported as one.
 * Adopt multi-level hash strategy for graphite searches. The goal here is to
   be faster and more memory-efficient, with a focus on memory efficiency.
 * Fix logging bug where long lines could end up running together.
 * Fix crash bug in histogram fetching API.

## Changes in 0.13.9
2018-10-16

 * Installer and startup wrapper will update ownership of `/opt/circonus/etc`
   and `/opt/circonus/etc/irondb.conf` to allow for automatic updating of the
   topology configuration during rebalance operations.
 * Performance improvements to parsing surrogate database at startup.
 * Fix some potential crashes.
 * Disable saving ptrace stdout output files in the default
   circonus-watchdog.conf file.

## Changes in 0.13.8
2018-10-12

 * Expose more jobq modification via console.
 * Fix wildcard/regex queries inside tag categories.
 * Fix issue where certian job queues could have concurrency
   of zero, causing deadlock.
 * Add activity ranges to tag_cats/vals.
 * Add category param to tag_vals.

## Changes in 0.13.7
2018-10-11

 * Documentation: fix missing rebalance state.
 * Add log deduplication to avoid spamming errorlog with
   identical messages.
 * Fix potential deadlock that could be triggered when forking
   off a process to be monitored by the watchdog.
 * Fix some potential crashes/memory leaks.
 * When loading a new topology, return 200 status instead of 500
   if the topology is already loaded.
 * Support tag removal.
 * Performance/stability improvements for activity list operations.

## Changes in 0.13.6
2018-10-01

 * Move Zipkin setup messages out of the error log and into the debug
   log.
 * Skip unparseable metric_locators during replication.
 * Turn off sync writes in tagged surrogate writer.
 * Fix potential crashes when check_name is NULL.

## Changes in 0.13.5
2018-09-25

 * Disable asynch core dumps by default.
 * Use the metric source for incoming metrics instead of hardcoding
   to RECONNOITER.
 * Fix some potential use-after-free crashes.
 * Fixed a crash where we would erroneously assume null termination.
 * Performance and correctness fixes to internal locking mechanism.
 * Fix some instances where we would potentially attempt to access a
   null metric name.

## Changes in 0.13.4
2018-09-21

 * Installer bug since 0.13.1 set incorrect ZFS properties on some datasets.
   New installs of 0.13.1 or later may need to run the following commands to
   restore the correct property values. Existing deployments that upgraded from
   version 0.13 or earlier were not affected.
   ```
zfs inherit -r quota <poolname>/irondb/data
zfs inherit -r quota <poolname>/irondb/nntbs
zfs inherit -r quota <poolname>/irondb/hist
zfs inherit -r quota <poolname>/irondb/localstate
zfs inherit -r quota <poolname>/irondb/logs
zfs inherit -r quota <poolname>/irondb/lua
zfs inherit -r quota <poolname>/irondb/metric_name_db
zfs inherit -r logbias <poolname>/irondb/redo
zfs inherit -r logbias <poolname>/irondb/text
   ```
 * Fix memory leaks and invalid access errors that could potentially
   lead to crashes.

## Changes in 0.13.3
2018-09-18

 * Fix hashing function for the reverse surrogate cache.
 * Fix loading of metrics db index when iterating surrogate entries
   on startup.
 * Improve logging for surrogate db when there are ID collisions.
 * Accept check name and source in /surrogate/put - do not allow duplicate
   surrogate ids in the cache.
 * Performance improvements to inter-node gossip and NNTBS data writing.
 * Allow purging metrics from in-memory cache.
 * Fix some potential crashes on unexpected data.
 * Allow using tag search to define retention period for metrics.

## Changes in 0.13.2
2018-09-13

 * Fixes for journal surrogate puts and activity rebuilds.
 * Fix bug where software would loop forever if journal writes
   were in the future.

## Changes in 0.13.1
2018-09-11

 * Various performance improvements.
 * Use progressive locks in surrogate DB.
 * Documentation: fix incorrect header name for raw data submission with
   Flatbuffer.
 * Allow deleting metrics by tag.
 * Allow deleting all metrics in a check.
 * Allowing deleting metrics based on a wildcard for NNT, text, or histogram
   data.
 * Allow 4096 chars for metric name ingestion
 * New CAQL function:
   * `group_by:*` package provides functions to aggregate metrics by tags

## Changes in 0.13
2018-08-15

 * **Service config change for EL7**: We now ship a native systemd service
   unit configuration, rather than a traditional init script. The unit name
   remains the same, but any configuration management or other scripting that
   used the `chkconfig` and `service` commands should be updated to use
   `systemctl`.
 * Installer: better validation of user input.
 * Config option to disable [Activity Tracking](/activity_tracking.md) which
   can cause write latency spikes at higher ingest volumes. A fix for this
   behavior will be coming in a future release.
   * Add an attribute, `activity_tracking="false"` to the
     `<surrogate_database>` line in `irondb.conf` to disable tracking.
   * Note that certain search parameters that depend on activity tracking will
     not work while tracking is disabled, and may not be accurate if tracking
     is reenabled after some time. Any search query that uses
     `activity_start_secs` or `activity_end_secs` will not work when tracking is
     disabled.
 * Memory leak fixes in Graphite result handling.
 * New CAQL functions:
   * `each:*` package provides functions that operate on all input slots at
     once. [CAQL Reference: package each](https://login.circonus.com/resources/docs/user/caql_reference.html#Packageeach)
   * `TopK` global function returns the top `k` streams over the current
     `VIEW_RANGE` using either a `mean` or `max` comparator. [CAQL Reference: global functions](https://login.circonus.com/resources/docs/user/caql_reference.html#GlobalFunctions)

## Changes in 0.12.5
2018-08-07

 * Crash fix on unparseable metric names
 * Journal fix in pre\_commit mmap space

## Changes in 0.12.4
2018-08-02

 * More memory leak fixes
 * Fixes for graphite tag support
 * Fix for greedy name matching in graphite queries
 * Support blank tag values
 * CAQL if statements and negation operators
 * CAQL optimizations
 * Support for building/rebuilding higher level rollups from lower level rollups
 * Rebalance adds a new completion state to fix races when finishing rebalance ops


## Changes in 0.12.3
2018-07-12

 * More memory leak fixes in name searches
 * Rebalance fixes
 * Embed a default license if one isn't provided
 * Support for [raw deletes](/api/delete-raw.md)

Documentation changes:
 * Add raw delete API

## Changes in 0.12.2
2018-07-09

 * Fix memory leak in name searches

## Changes in 0.12.1 (unreleased)
2018-07-09

 * Enable heap profiling

## Changes in 0.12
2018-07-05

This release brings several major new features and represents months of hard
work by our Engineering and Operations teams.

 * New feature: [Stream Tags](/tags.md)
   * These are tags that affect the name of a metric stream. They are
     represented as `category:value` pairs, and are [searchable](/api/search-tags.md).
   * Each unique combination of metric name and tag list counts as a new metric
     stream for licensing purposes.
 * New feature: [Activity Tracking](/activity_tracking.md)
   * Quickly determine time ranges when a given metric or group of metrics was
     being collected.
 * New feature: Configurable [rollup retention](/configuration.md#nntbs) for numeric data.
   * Retention is per rollup period defined in configuration.
 * Operations: There is a one-time operation on the first startup when
   upgrading to version `0.12`.
   * As part of Stream Tags support, the
     [metric\_name\_database](/configuration.md#metricnamedatabase) has been
     combined with another internal index and is no longer stored separately on
     disk.
   * The metric name database was always read into memory at startup. After the
     one-time conversion, its information will be extracted from the other index
     on subsequent startups. The time to complete the conversion includes the
     same amount of time to read the existing metric name database as well as
     to write out an updated index entry for each record encountered.
     Therefore, it is proportional to the number of unique metric streams
     stored on this node.
 * Operations: The `raw_database` option [rollup_strategy](/configuration.md#rawdatabase-rollupstrategy)
   now defaults to `raw_iterator` if not specified.
   * If upgrading with a config that does not specify a `rollup_strategy`, an
     active rollup operation will start over on the timeshard it was
     processing.
 * Operations: Add the ability to [cancel a sweep delete](/api/sweep-delete-cancel.md)
   operation.
 * Operations: Remove the reconstitute-reset option (`-E`) and replace with a
   more complete solution in the form of a script, `reset_reconstitute`, that
   will enable the operator to remove all local data and start a fresh rebuild.
 * CAQL: add methods `time:epoch()` and `time:tz()`
 * Installer: use default ZFS recordsize (128K) for NNT data. This has been
   shown experimentally to yield significantly better compression ratios.
   Existing installations will not see any change. To immediately effect these
   changes on an existing install, issue the following two commands:
   ```
zfs inherit -r recordsize <pool>/irondb/data
zfs inherit -r recordsize <pool>/irondb/nntbs
   ```
   where `<pool>` is the zpool name. Users of versions < [0.11.1](#changes-in-0111)
   can omit the second command (this dataset will not be present.) The
   recordsize change only affects new writes; existing data remains at the
   previous recordsize. If the full benefit of the change is desired, a
   [node rebuild](/rebuilding-nodes.md) may be performed.
 * Documentation: Raw Submission API documentation for already required
   X-Snowth-Datapoints header
 * Documentation: Text and Histogram deletion APIs were out of date.
 * Documentation: Update formatting on API pages, which were auto-converted
   from a previous format.
 * Performance and stability fixes too numerous to list here, though there are
   some highlights:
   * Converted UUID handling from libuuid to libmtev's [faster implementation](http://circonus-labs.github.io/libmtev/apireference/c.html#u).
   * Optimized replication speed.

## Changes in 0.11.18
2018-04-12

 * Fix a bug causing unnecessary duplicated work during sweep deletes

## Changes in 0.11.17
2018-04-10

 * Fix for http header parsing edge case

## Changes in 0.11.16
2018-04-09

 * Allow control over max ingest age for graphite data via config
 * Optionally provide graphite find and series queries as flatbuffer data
 * Fix epoch metadata fetch for NNTBS data
 * Reconstitute state saving bug fixes
 * Fix cleanup of journal data post replication

Documentation changes:
 * Add hardware selection advice and system profiles
 * Correct color rules for latency summaries
 * Various small doc fixes

## Changes in 0.11.15
2018-03-23

 * Fix potential use-after-free in raw numeric fetch path.
 * Various fixes to NNTBS batch conversion.
 * Crash fixes when dealing with NNTBS shards.
 * UI changes for Replication Latency display:
   * Initially all remote node latencies are hidden, with just the heading
     displayed. Click on a heading to expand the remote node listing.
   * A node's average replication latency is now displayed at the right end of
     the heading, and color-coded.
 * Disable Lua modules when in reconstitute mode.
 * Don't hold on to NNT filehandles after converting them to NNTBS.

Documentation changes:
 * Include files and Lua modules.
 * New UI replication tab display.

## Changes in 0.11.14
2018-03-13

 * Fix bug in NNT reconstitution

## Changes in 0.11.13 (unreleased)
2018-03-12

 * Fix for throttling during reconstitute operations
 * Several small fixes and cleanups

## Changes in 0.11.12
2018-03-08

 * Add an offline NNT to NNTBS conversion mode.
   * Default conversion is "lazy", as NNT metrics are read.
   * For read-heavy environments this may produce too much load, so the offline
     option can be used to take one node at a time out of the cluster and
     batch-convert all its NNT files to NNTBS block storage.
 * Performance improvements to gossip replication, avoids watchdog timeout in
   some configurations.
 * Fix several crash bugs in reconstitute, NNTBS, and journaling.
 * Silence noisy error printing during NNTBS conversion.
 * Formatting fix to a gossip error message (missing newline).

Documentation changes:
 * Add NNTBS dataset to reconstitute procedure.
 * New NNTBS conversion-only operations mode (`-N`).
 * Clarify that in split clusters, write copies are distributed as evenly as
   possible across both sides.
 * Show the gossip age values that lead to green/yellow/red display in the
   Replication Latency UI tab.

## Changes in 0.11.11
2018-02-23
 * Final deadlock fixes for timeshard management
 * Protect against unparseable json coming back from proxy calls

## Changes in 0.11.10
2018-02-22
 * More deadlock fixes for timeshard management

Documentation changes:
 * Note the lazy migration strategy for NNT to NNTBS conversion.

## Changes in 0.11.9
2018-02-20
 * Fix deadlock that can be hit when attempting to delete a shard during heavy
   read activity.
 * Use new libmtev `max_backlog` API to shed load under extreme conditions.
 * Internal RocksDB tuning to reduce memory footprint, reduce file reads and
   improve performance.
 * Add a tool to repair the raw DB if it gets corrupted, as with an unexpected
   system shutdown.

Configuration changes:
 * Add a "startup" log to shift certain initialization logs out of the
   errorlog.
   * Reduces clutter and makes it easier to see when your instance
     is up and running.
   * New installs will have this log enabled by default, written to
     `/irondb/logs/startuplog` and rotated on the same policy as `errorlog`.
   * To enable on an existing installation, add this line to
     `/opt/circonus/etc/irondb.conf`, in the `<logs>` stanza (on a single
     line):
     ```
<log name="notice/startup" type="file" path="/irondb/logs/startuplog"
timestamps="on" rotate_seconds="86400" retain_seconds="604800"/>
     ```

Documentation changes:
 * Appendix with cluster sizing recommendations.
 * GET method for `sweep_delete` status.

## Changes in 0.11.8
2018-02-09
 * Minor fix to reduce error logging

## Changes in 0.11.7
2018-02-08
 * Minor fixes for histogram database migration

Documentation changes:
 * Add new section on `nntbs` configuration

## Changes in 0.11.6
2018-02-08
 * NNTBS timesharded implementation
 * Changes for supporting very large reconstitution
 * Do raw database reconstitution in parallel for speed

Documentation changes:
 * Add new section on the `sweep_delete` API, useful for implementing retention
   policies
 * Add new section on migrating to a new cluster from an existing one.
 * Add page documenting `snowthd` command-line options.

## Changes in 0.11.5
2018-01-23

 * Yield during reconstitute/rebalance inside NNTBS to prevent starvation of other ops

## Changes in 0.11.4
2018-01-22

 * Fix for iterator re-use in error edge case

## Changes in 0.11.3
2018-01-22

 * Safety fix for rollup code
 * Corruption fix on hard shutdown or power loss

## Changes in 0.11.2
2018-01-18

 * Crash fix for rollup code
 * Lock fix for conversion code
 * **Changes for new installations** - new installations will have different defaults
   for `<raw_database>` settings:
	* `granularity` goes from 1 day to 1 week.
	* `min_delete_age` goes from 3 days to 4 weeks.
	* `delete_after_quiescent_age` goes from 12 hours to 2 hours.
	* `rollup_strategy` was added.
   It is fine to mix new nodes installed with these settings with older
   nodes who have the older settings.  It is *not* fine to change these
   settings on an existing installation.

Documentation changes:
 * Describe `rollup_strategy` in the `<raw_database>` config

## Changes in 0.11.1
2018-01-18

 * Fixes for NNTBS
 * Add NNTBS stats to admin UI
 * Various smaller fixes

## Changes in 0.11
2018-01-12

 * Store rollup data in a new format yielding better performance on insert and rollup (NNTBS)
 * Performance improvements for lua extensions
 * Reduce logging to error sink
 * Many smaller fixes and improvements
 * Dropped support for OmniOS (RIP)

## Changes in 0.10.19
2017-12-18

 * Improve rollup speed by iterating in a more natural DB order, with
   additional parallelization.
 * The `setup-irondb` script will now log its output, in addition to stdout. It
   will log to `/var/log/irondb-setup.log` and if run multiple times will keep
   up to five (5) previous logs.
 * The [snowthimport](installation.md#import-topology) tool will now fail
   with an error if the topology input file contains any node IDs with
   uppercase letters.

Documentation changes:
 * Note that all supplied UUIDs during initial setup and cluster configuration
   should be lowercase. If uppercase UUIDs are supplied, they will be
   lowercased and a warning logged by setup.

## Changes in 0.10.18
2017-12-06

 * Fix crash in fair queueing
 * Finish moving rollups to their own jobq

## Changes in 0.10.17
2017-12-05

 * Restore fdatasync behavior from rocksdb 4.5.1 release
 * Move rollups to their own jobq so as to not interfere with normal reads
 * Implement fair job queueing for reads so large read jobs cannot starve out other smaller reads

## Changes in 0.10.16
2017-11-27

 * New rocksdb library version 5.8.6

## Changes in 0.10.15
2017-11-21

 * More aggressively load shed by forcing local data fetch jobs to obey timeouts

## Changes in 0.10.14
2017-11-20

 * Allow config driven control over the concurrency of the data\_read\_jobq
 * Short circuit local data read jobs if the timeout has elapsed
 * Add all hidden stats to internal UI tab

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

## Changes in 0.8.11
2017-04-05

 * Speed up reconstitute through parallel processing.

## Changes in 0.8.10
2017-04-04

 * Improve throughput via socket and send-buffer tuning fixes.
 * Fix watchdog timeouts when reloading large metric databases.

## Changes in 0.8.9
2017-04-03

 * Preserve null termination in metric names for proper duplicate detection.

## Changes in 0.8.8
2017-03-31

 * Turn off gzip in reconstitute, as testing shows throughput is better without
   it.
 * Avoid performing rollups or deletions on a reconstituting node.
 * Memory leak fixes.

## Changes in 0.8.7
2017-03-24

 * Performance fixes for reconstitute.
 * Memory leak fixes.

## Changes in 0.8.6
2017-03-21

 * Fix internal wildcard queries, and limit Graphite metric names to 256
   levels.

## Changes in 0.8.5
2017-03-17

 * Build Graphite responses using `mtev_json` instead of custom strings.

## Changes in 0.8.4
2017-03-14

 * Set a maximum metric name length on ingestion.

## Changes in 0.8.3
2017-03-10

 * Various replication fixes.
 * Fixes for parsing errors and startup crashes.

## Changes in 0.8.2
2017-03-01

 * Reject Graphite metrics with an encoded length greater than 255.

## Changes in 0.8.1
2017-02-27

 * Internal testing fixes.

## Changes in 0.8
2017-02-27

 * De-duplicate proxied requests.
 * Deal with unparseably large number strings.

## Changes in 0.7
2017-02-23

 * Add raw ingestion.
 * Stricter Graphite record parsing.
 * Memory leak and header-parsing fixes.

## Changes in 0.6
2017-02-15

 * Better handling of JSON parse errors during reconstitute.
 * Enable `Accept-Encoding: gzip`, compress outgoing replication POSTs with
   lz4f.
 * Optimize UUID comparison to speed up reconstitute.

## Changes in 0.5
2017-01-31

 * Fix crash from Graphite listener connection handling.
 * Refactor text metric processing in preparation for raw database.

## Changes in 0.4
2017-01-16

 * Fix rollup span calculation for Graphite fetches.
 * Support getting the topology configuration from an included config file.

## Changes in 0.3
2016-12-29

 * Allow reconstituting of individual data types.
 * UI fixes for displaying licenses.
 * Memory leak, crash and hang fixes.

## Changes in 0.2
2016-11-29

 * Don't recaclulate `counter_stddev` when counter in `NaN`.

## Changes in 0.1
2016-11-29

 * Add Graphite support.

## Changes in 0.0.2
2016-11-21

 * Fix issues with various inputs being `NaN`.

## Changes in 0.0.1
2016-11-17

 * Initial version. Start of "IRONdb" branding of Circonus's internal TSDB
   implementation.
