# Changelog

## Changes in 0.19.1
2019-12-17

 * Fix memory leaks in NNTBS and raw reconstitute paths.

## Changes in 0.19.0
2019-12-10

 * Change NNTBS reconstitute to iterate through entire shards rather than pulling individual
   metrics.
   THIS IS A BREAKING CHANGE - any reconstitute that is in progress when this deploys
   will need to be restarted from the beginning. All nodes will need to be brought up to
   the latest version as well.
 * Change framing of raw reconstitute data to improve efficiency.
 * CAQL: Add base parameter to the integrate() function.
 * CAQL: Add histogram:subtract() function
 * [libmtev 1.9.8](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#198)

## Changes in 0.18.8
2019-11-21

 * Fix infinite loop when `/fetch` exhausted its deadline and nodes are down.
 * Make the `resize_cluster` script load the new topology on removed nodes.
 * Fix bug in flatbuffer byte alignment where the code was inaccurately determining
   if we needed additional byte alignment.
 * Support `count_only=1` for `find//tags`.
 * Align and validate all surrogate flatbuffer data before attempting to use it.
   This will prevent using incorrect values and/or crashing on bad data.
 * Fix bug with metric type changes using surrogate put REST API.

## Changes in 0.18.7
2019-11-18

 * Fix crash when fetching histograms with a period less than 1 second
 * Always adjust Graphite step to best NNT rollup if no raw data found
 * Add new log stream for Graphite step adjustments (`debug/graphite/step_adjust`)
 * CAQL: Fix a bug with handling missing data in diff()
 * CAQL: Improve performance of window:/rolling:/aggregate: functions in #strict mode
 * CAQL: Add `aggregate:*` package for controlling data aggregation on graphs
 * CAQL: Support grouping by multiple tags
 * CAQL: Performance improvements to `diff()`/`integrate()`/`delay()`/`is_missing()`
 * [libmtev 1.9.5](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#195)

## Changes in 0.18.6
2019-11-08

 * Fix potential null dereference/crash when iterating raw database during reconstitute
 * Fix crash in reconstitute where attempting to defer rollups until after the reconstitute was
   finished was causing a race leading to a crash.
 * CAQL: Add multiple input slots to the `delay()` function and improve its performance
 * CAQL: Add deprecation warnings to `histogram:window` and `histogram:rolling`;
   `window:merge` and `rolling:merge` should be used instead.
 * CAQL: Revise the time aggregation functions `window:*` and `rolling:*`
   - Improve performance by leveraging pre-aggregated data
   - By default the results of `window:*` functions no longer lag behind the incoming data
     when displayed on graphs. The old behavior can be restored by passing `align="end"` as
     a parameter.
   - Add support for multiple input streams
   - Align window boundaries consistently
   - Add `window:first()` function that selects the first sample in each window
   - Add `window:merge()` function to aggregate histograms over time
   - Add `skip` parameter to control the advancement of time windows
   - Add `period` parameter to control the granularity of input data
   - Add `align=start/end` parameter to control alignment of the output data
   - Add `offset` parameter to control window offset against UTC
 * CAQL: Add #strict directive that forces serial data processing with period=1M
 * CAQL: Improve speed and accuracy of the `integrate()` function.

## Changes in 0.18.5
2019-10-29

 * Fix surrogate/put `type` setting.
 * Prefer `uuid` and `caetgory` as fields instead of `check_uuid` and `source` to match the /find output.
 * Disable `nnt_cache` module in the default configuration for simplicity.
 * Make use /fetch deadline handlers in lua/CAQL
 * Support X-Snowth-Timeout and X-Snowth-Deadline headers to /fetch
 * Allow activity=0 to /find//tags to suppress activity information.
 * Support telnet-like console access via the administrative web UI.
 * CAQL: Add deprecation warning to search:metric() and metriccluster() function.
   Search v2 and Metric-clusters have been deprecated for a while now.
   We plan to remove these deprecated function in 2020-01-31.
   This will affect caql checks as well as CAQL Datapoints on graphs.
   With this change, the UI will show users a warning, when one of those deprecated functions is used.
   Circonus offers the more-powerful tag-search feature, exposed as find() in CAQL.
 * CAQL: Add default labels to `histogram:*` output
 * CAQL: Restrict sorting of results to the find() function, so that, e.g. top-k output is not sorted by label
 * CAQL: Add tag:remove() function
 * CAQL: Set default/max limits for CAQL find() queries to 1000/3000 (configurable)
 * CAQL: Speed-up data fetching with the metric(), and the deprecated search:metric() and metriccluster()
   functions, by leveraging the /fetch endpoint.
 * CAQL: Fix bugs with limiting and sorting outputs. Introduced: 2019-10-22
 * CAQL: Optimize a number of query patterns to leverage federated data processing:
   - find() | stats:{sum,mean}
   - find() | count()
   - find() | top()
   - find:histogram() | histogram:merge()
   - find:histogram() | histogram:sum() | stats:sum()
 * CAQL: Fix count() function to not count NaN values
 * [libmtev 1.9.2](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#192)


## Changes in 0.18.4
2019-10-16

 * Support trailing \*\* in graphite queries in a way that is leaf-only.
 * Support a filter config option for the monitor module.
 * Support histogram input for /fetch `groupby_stats`.
 * Implement histogram /fetch transforms: `{inverse_,}{quantile,percentile}`
   and `count_{above,below}`.
 * Bug: Fix crashes related to bad locking when adding/removing a metric locator
   from the surrogate cache.
 * Bug: Fix potential integer overflow when using the `/fetch` endpoint that could
   cause occasional incorrect results.
 * CAQL: Fix account_id handling for histogram summary views.
 * CAQL: Add sensible default labels to histogram:percentile() output.
 * CAQL: Performance improvements to integrate() function.
 * CAQL: Leverage /fetch endpoint for find() operations. This is a significant
   performance improvement that should make CAQL find() operations much
   faster.

## Changes in 0.18.3
2019-10-07

 * Support `__activity:start-end` inside search query nodes.
 * Prefix accelerate ART-based tag searches with escaped special characters
   (`/^foo\.bar\.baz\.[^a]*cpu_*/` would previously prefix only `foo`, but will
   now prefix `foo.bar.baz.`)
 * Performance improvements for raw data reconstitute.
 * Bug: Fix potential stack smash when writing items to the surrogate cache.
 * [libmtev 1.8.5](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#185)

## Changes in 0.18.2
2019-10-01

 * Performance improvements releated to opening raw timeshards.
 * Disable filesystem read-ahead on NNTBS shards to improve performance.
 * Various Performance improvements related to data fetching:
   * Less piecemeal work is performed, which means that long runs of fetches
     are performed in the same jobq and not fanned out as extensively.
   * epoch/apocalypse times for numeric fetches are accelerated using activity
     tracking.
   * The /rollup engine=dispatch endpoint now does a simple merge of nntbs and raw.
   * Legacy /rollup behaviour of a complex nntbs/raw/nntbs sandwich is available
     via engine=dispatch_coarsen.
 * Greatly improve performance when fetching rollup data for a stream that
   has no historic data before the starting time and for which there are many prior
   raw timeshards. This improves the fetch time from tens of seconds to tens
   of milliseconds.
 * The graphite series fetch functions no longer move the `from` parameter forward
   to limit leading nulls in output.
 * Bug: Fix memory leaks in raw data iterator and surrogate db loading
 * Bug: Change the /fetch API endpoint to perform work in the snowth_fetch_remote
   and snowth_fetch_local jobqs. It was using an incorrect jobq before.
 * Bug: Fix use-after-free that could cause crashes when using the /fetch API
   endpoint.
 * Bug: Fix crash in graphite fetching when there are greater-than or equal-to
   the data replication value (W) nodes down.
 * Bug: Fix ck_fifo usage to prevent memory misuse that could lead to crashes
   when loading the surrogate DB or processing journal replication data.
 * Bug: Fix various potential crashes in reconstitute/rebalance.
 * Bug: Fix console web UI to prevent abusive loading of json data after a suspended
   connection is reestablished.
 * Bug: Replace confusing graphite fetch error messages with more coherent ones.
 * [libmtev 1.8.2](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#182)

## Changes in 0.18.1
2019-09-24

 * Change raw data reconstitute to use flatbuffers instead of M records. This will
   require all nodes in the cluster to be updated before reconstitute will work
   properly.
 * Add `surrogate_database/@{latest_future_bound,implicit_latest}` and track the
   latest arriving value for metrics accordingly.  Expose them via find according
   to a `latest` query string parameter.
 * Add ability to enable/disable the NNT Cache module via a POST command
   (/module/nnt_cache?active={0,1})
 * Add ability to manually flush the NNT Cache via a POST command
   (/module/nnt_cache/flush)
 * Performance improvements in database iteration - should improve
   both insert and fetch operations.
 * Support `**` wildcard expansion in Graphite find queries.
 * Bug: Ensure that all NNTBS data is transferred correctly during certain reconstitute
   edge cases, such as when the NNTBS metric was the final metric in a shard or if there
   are long gaps where there is no data for a metric resulting in the data not being
   stored in contiguous shards.
 * Bug: Ensure that surrogate db reconstitute is finished before inserting text
   and histogram records during reconstitute to avoid potential race conndition
   when updating the surrogate db.
 * CAQL: Support for labeling multiple output streams with label() function.
 * [libmtev 1.8.0](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#180)


## Changes in 0.18.0
2019-08-27

 * Remove outdated/broken `/activate` endpoint
 * Add additional safety to the topology compilation progress - fail to compile a topology
   if the write_copies value is higher than the number of nodes.
 * During data fetch, if no raw data is present, Graphite rollup span now aligns to the best
   NNT rollup available.
 * Improve performance, scale, and versatility of rebalance operations.
 * Bug: Fix broken topology change rejournal code - was writing data to ourselves
   pointlessly and was occasionally writing data with a bad topology.
 * Bug: Reject incoming data puts when node is ephemeral or not participating
   in current topology. Previously, this would cause crashes.
 * CAQL: Make use of activity period tracking to avoid fetching empty metrics.
 * [libmtev 1.7.0](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#170)

## Changes in 0.17.3
2019-08-15

 * Performance improvements to inter-node data journaling.
 * Bug: Fix prometheus module label equality searches for values beginning
   with `/` or containing wildcard expansions `*` and `?`.
 * Bug: Fix bug in reconstitute where the reconstituting node was not
   writing correct check name and account id data to the surrogate db
 * Bug: Fix crash when fetching raw numeric data using metric names that
   cannot be canonicalized.
 * [libmtev 1.6.26](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1626)

## Changes in 0.17.2
2019-07-29

 * Add ability to use hostnames in cluster topology files - previously, only IP
   addresses were allowed.
 * Improve performace by not updating indexes on non-metadata surrogate DB writes.
 * Bug: Fix Graphite sum egress function - the fetch was erroneously summing data 
   that was already summed, resulting in reporting values that were larger than
   expected.
 * CAQL: Fix a bug in find() where fully completed queries would be reported as
   truncated
 * CAQL: Don't truncate find() queries that have been running for less than 4 seconds.
 * [libmtev 1.6.24](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1624)

## Changes in 0.17.1
2019-07-18

 * Bug: Various memory leaks fixed in the /fetch endpoint.
 * Allow snowth topologies to use names instead of just IPv4 addresses in
   the address attribute, they are resolved once at runtime compilation.
 * Bug: Fix external metadata replication getting stuck in a loop due to
   improper checkpoint parsing.
 * [libmtev 1.6.21](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1621)

## Changes in 0.17.0
2019-07-16

 * Prometheus and OpenTSDB integrations are now active by default for new
   installations. If you previously activated one or both of these modules in
   `/opt/circonus/etc/irondb-modules-site.conf`, you may remove those
   configurations at your convenience after upgrading, though it will not be an
   error for the module to be configured more than once.
 * Dump out query text to error log on a parse error with tag query finds.
 * Fix clustered reads in the prometheus module.
 * Increase default concurrency in process_journal and data_read jobqs.
 * Implement activity tracking for graphite queries.
 * Improve surrogate database loading speed.
 * Bug: Fix occassionaly crashes in pipelined replication journal receptions.
 * Bug: Optimize surrogate replay and prevent/repair corruption via auto-repair.
 * Bug: Fix condition where surrogate checkpoint would not complete if no
   surrogate activity has transpired since boot. This fixes many issues that were
   caused by this, such as deletes and raw data rollups getting stuck and not
   completing.
 * Bug: Fix issue where we'd occasionally return null data when doing a proxy
   using the rollup endpoint.
 * Bug: Fix memory leaks related to activity tracking.
 * CAQL: Return an error on find calls if no account information is found.
 * [libmtev 1.6.20](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1620)

## Changes in 0.16.3
2019-06-26

 * Add activity data to `tags/<id>/find` JSON responses.
 * Bug: Address inconsistent activity windows on single stream batch loading.
 * Bug: Fix consistency issue with in-memory indices of check/tag set-crdt data.
 * Bug: Fix potential crashes related to not acquiring the read lock before
   cloning an oil (ordered interval list) object for activity tracking.
 * Bug: Fix memory leaks that occur in the metrics database when using `find`
   to search for metrics.
 * [libmtev 1.6.16](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1616)

## Changes in 0.16.2
2019-06-19

 * Change default text fetching to provide the prior value if the requested
   start offset is between recorded samples. Expose `lead=<true|false>`
   query string parameter, defaulting to true, to turn this feature on
   or off.
 * Bug: Fix crash on error in full delete with long metric names and tags.
 * Bug: Remove erroneous "missing activity cf" message in log on startup.
 * Bug: Remove temporary files accidentally left in /var/tmp during
   reconstitute.
 * Bug: Fix opentsdb parsing bug where we handled timestamps without
   decimal points incorrectly.
 * CAQL: Update docs.
 * [libmtev 1.6.14](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1614)

## Changes in 0.16.1
2019-06-04

 * Bug: Prevent null pointer exception in the data replication path when the
   check name is undefined.
 * CAQL: Assert that start times are before or equal to end times in
   queries.

## Changes in 0.16.0
2019-05-28

 **WARNING: Downgrades will not be possible once this version is
    installed**

 * Introduce a dedicated column family in the surrogate database to track
   activity. This results in reduced I/O workload.
 * Change histogram quantile/sum/mean operations to return approximations
   that minimize the relative error.
 * Non-histogram monitor metrics should be tracked as numeric or text,
   not histogram.
 * Ensure `/find` endpoints emit valid JSON.
 * Faster setmeta serialization for merge.
 * Increase default `surrogate_writer` job queue concurrency to `6` (from `1`).
 * Fix race in metrics db (search indexes) where some metrics might be
   omitted during index construction.
 * Fix crash when `/rollup` rollup_span == `0` (and require rollup_span > `0`).
 * Documentation: add [Monitoring](/monitoring.md) page describing how to
   obtain and optionally auto-store internal node statistics.
 * Bug/CAQL: Fix histogam:count_below() to also count samples in the current bucket,
   as the documentation states.
 * Bug/CAQL: histogram:stddev() will now return nan ("not a number") for histograms
   with a single value instead of `0`.
 * [libmtev 1.6.12](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1612)

## Changes in 0.15.8
2019-05-09

 * /rollup/ and CAQL fetching functions now correctly defer reads on
   replication delay.
 * Incoming rest calls are now assigned task IDs based on either the
   X-Snowth-TaskId header or an an active zipkin trace id.
 * Performance improvements when debugging is disabled.
 * Allow graphite and opentsdb raw socket to accept tags with special
   characters.
 * Add surrogate checkpoint latency stats.
 * Added an optional header, "x-snowth-metadata-flush", to delete
   requests. If set to `0`, this will disable metadata flushing.
 * [libmtev 1.6.10](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1610)

## Changes in 0.15.7
2019-05-01

 * CAQL: Fix regression introduced in version 0.15.6 that would cause some
   CAQL fetches to fail.

## Changes in 0.15.6
2019-04-30

 * Fix a performance regression introduced by 0.15.5 where CPU usage
   could spike.
 * Performance improvements when looking up locations on the topology
   ring.
 * Ensure all journal replication threads are supplied with work. Previously,
   if more than one replication thread existed and there was not sufficient
   load to utilize all of them, some journal segments were not removed after
   their data was replicated. This led to increased disk usage over time, and
   was exacerbated by a change to the default journal replication concurrency
   in 0.15.3.
 * CAQL: Add type checking facilities to CAQL function arguments.
 * CAQL: histogram:count_*() processing on higher periods, was off by a factor
   of VIEW_PERIOD/60. This is corrected now.
 * CAQL: Expand label() functionality.
 * CAQL: Add tag() function.

## Changes in 0.15.5
2019-04-23

 * Fix max_ingest_age and max_clock_skew parameters in graphite handling.
   max_clock_skew will default to the raw db max_clock_skew or else one
   day. Records will be elided if they are earlier than now - max_ingest_age
   or later than now + max_clock_skew.
 * Fix thread safety issues that could lead to occasional crashes.
 * CAQL: Fix `find:histogram_cum()` functionality.
 * CAQL: Performance Improvements.
 * Documentation: Add docs on the [UI Internals tab](/operations.md#internals-tab),
   which contains a rich set of statistics for troubleshooting performance problems.
 * [libmtev 1.6.9](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#169)

## Changes in 0.15.4
2019-04-12

 * Fix startup crash bug in maintaining retention windows.
 * Fix reconstitute bug in cases of incomplete file reads.
 * Fix bug where multiple time retention maintenance jobs could run
   concurrently.
 * Performance improvements to inter-node gossip communications.
 * Support FlatBuffers requests in /histogram read endpoint.
 * Support backlog display and stats filtering in UI.
 * OpenTSDB ingestion is now an optional module.
 * CAQL: Increase the default histogram fetch limit to 3M.
 * CAQL: Accelerate sum/sub/prod/div operations.
 * CAQL: histogram:percentile and histogram:count_* operations now act on multiple
   input slots rather than just the first one.
 * Documentation: put `gpgcheck=0` back into crash-reporting repo stanza for
   EL7. These packages are not produced by Circonus, we simply mirror them.
 * [libmtev 1.6.8](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#168)
 

## Changes in 0.15.3
2019-04-02

 * Limit search results to 10,000 items by default. This can be overridden by
   setting a request header, `x-snowth-advisory-limit`, to a positive integer
   value. Setting it to -1 or "none" removes the limit.
 * Change default [journal replication concurrency](/configuration.md#journal-replicateconcurrency)
   from 1 to 4.
 * Memory leak and crash fixes.
 * Alter search to include check_tags if present.
 * Add flag to allow nodes to rebalance in parallel rather than forcing nodes
   to rebalance one at a time.
 * Various performance improvements.

## Changes in 0.15.2
2019-03-27

 * Improved the CAQL label function to support name and tag extraction
 * Faster surrogate writes (adding new metrics and updating activity
   information)
 * Improve NNTBS timeshard open/close performance by reducing unnecessary
   locking
 * Support added for cumulative histograms at read time
 * Make rebalance more robust
 * Reduce graphite read workload on datasets with large timespans
 * Add native prometheus read/write endpoints
 * Fix crash under repetitive license violations

## Changes in 0.15.1
2019-03-19

 * Add module to monitor IRONdb statistics internally and feed
   them back into the DB.
 * [libmtev 1.6.5](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#165)

## Changes in 0.15.0
2019-03-18

 * Add support for OpenTSDB data ingestion.
 * Add eventer callback names for events. This will aid in
   debugging if zipkin spans are enabled and collected.
 * Remove support for untagged surrogates and surrogate migration.
 * Add support for pulling tagged stats by adding a "format=tagged"
   querystring to the stats.json API endpoint.
 * Move error log "file already found" from snowthimport binary
   to the debug log.
 * Fix typo in statistics: "hits_meta" is now "hit_meta".

## Changes in 0.14.18
2019-03-12

 * Support caching metric metadata in NNT cache.
 * Fix potential crashes and deadlocks in NNTBS timeshard open/close
   code.
 * Move graphite fetching code into a loadable module.
   * **If you are upgrading a node that was initially installed with a version
     prior to 0.13, ensure that you have the necessary config files included
     from `/opt/circonus/etc/irondb.conf`:**
       * Remove the `<eventer>` stanza near the top of the file.
       * Add the following two lines just below the include of `licenses.conf`:
         ```
  <include file="irondb-modules.conf" snippet="true"/>
  <include file="irondb-eventer.conf" snippet="true"/>
         ```

## Changes in 0.14.17
2019-03-11

 * Make efficiency changes to internal locking mechanisms to improve
   CPU utilization.
 * Fix bug where metadata deletions could break in-memory indexes.
 * Add optional NNTBS data cache to improve performance and reduce database
   iterations.
 * Installer: Create "metadata" directory and configuration setting. This
   directory is not currently used in standalone IRONdb installations.
 * [libmtev 1.6.4](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#164)

## Changes in 0.14.16
2019-02-25

 * Fix bug in node proxy code that caused incorrect timeout values to be used.
 * Fix various issues regarding using timeouts incorrectly during graphite data
   fetches.
 * Fix memory leaks that could occur during graphite error cases.

## Changes in 0.14.15
2019-02-20

 * Add optional metric prefix parameter to /tag_cats and /tag_vals APIs.
 * [libmtev 1.6.3](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#163)

## Changes in 0.14.14
2019-02-15

 * Node will now log error and exit when writes to rocksdb fail -
   previously, it would log the message and continue running, which
   could lead to data loss.
 * Fix off-by-one area in internal metric data storage struct that
   could cause potential crashes.
 * Added support for FlatBuffer requests to the `/graphite/tags/find` endpoint,
   which will greatly improve performance for users using Graphite 1.1.
 * Fix license expiration date display bug on GUI.
 * [libmtev 1.6.2](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#162)

## Changes in 0.14.13
2019-02-07

 * Fix stats and dashboard for NNTBS data
 * Enhance snowthsurrogatecontrol to dump all fields, as well as
   reverse or deleted records.
 * Fix various bugs that could result in crashes or deadlocks.
 * Various performance improvements.
 * Improvements to Graphite tag search - respect Graphite name hierarchy in
   search results.
 * [libmtev 1.6.1](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#161)

## Changes in 0.14.12
2019-01-17

 * Fix proxy bug in the `/find` API where certain proxy calls
   were being truncated, leading to incomplete results.
 * Added each:sub(x) and each:exp(x) operators to CAQL.
 * Performance improvements to full metric delete.
 * Deduplicate surrogate IDs from the database on
   startup.

## Changes in 0.14.11
2019-01-08

 * Fix bug where tagged metrics were not being loaded into the
   surrogate cache at startup correctly.
 * Tune the surrogate asynch update journal settings to improve
   performance.

## Changes in 0.14.10
2018-12-24

 * Eliminate raw delete timeout.
 * Fix bugs in surrogate DB serialization and add additional key
   validation on deserialization.

## Changes in 0.14.9
2018-12-17

 * Two related bug fixes in the surrogate DB that manifest with metrics
   whose total stream tag length is more than 127 characters. Metrics
   with such tag sets could appear to be missing from search results.
   Metrics that do not have any stream tags, or whose total tag set is
   less than 127 characters, are not affected.
 * Performance improvements to full delete.
 * Fix a bug that could cause crashes during reconstitute.

## Changes in 0.14.8
2018-12-13

 * Add optional metric delete debugging.
 * Fix bug that causes hanging when trying to delete
   certain metrics.
 * Fix occasional crash related to reading NNTBS data.

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
 * [libmtev 1.5.28](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1528)

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
 * [libmtev 1.5.26](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1526)

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
 * [libmtev 1.5.23](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1523)

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
 * [libmtev 1.5.19](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1519)

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
 * [libmtev 1.5.12](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1512)

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
 * [libmtev 1.5.11](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#1511)

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
 * [libmtev 1.5.7](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#157)

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
 * [libmtev 1.5.5](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#155)

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
 * [libmtev 1.4.5](https://github.com/circonus-labs/libmtev/blob/master/ChangeLog.md#145)

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
