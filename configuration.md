# IRONdb Configuration Options

IRONdb is implemented using
[libmtev](https://github.com/circonus-labs/libmtev/), a framework for building
high-performance C applications. You may wish to review the libmtev
[configuration documentation](http://circonus-labs.github.io/libmtev/config/)
for an overview of how libmtev applications are configured generally.

This document deals with options that are specific to IRONdb, but links to
relevant libmtev documentation where appropriate.

Default values are those that are present in the default configuration produced
during initial installation.

## irondb.conf

This is the primary configuration file that IRONdb reads at start. It includes
additional configuration files which are discussed later.

### snowth

```
<snowth lockfile="/irondb/logs/snowth.lock" text_size_limit="512">
```

IRONdb's libmtev application name. This is a required node and **must not** be
changed.

#### snowth lockfile

Path to a file that prevents multiple instances of the application from running
concurrently. You should not need to change this.

Default: `/irondb/logs/snowth.lock`

#### snowth text_size_limit

The maximum length of a text-type metric value. Text metric values longer than
this limit will be truncated.

Default: 512

> Text-type metrics are supported in IRONdb but Graphite currently has no way
> to render these when using a Storage Finder plugin.

### eventer

```
<include file="irondb-eventer.conf" />
```

Libmtev eventer system configuration is included from a separate file controlled
by the vendor. Changes to this file will be overwritten by package updates.

Details about the included configuration can be found in the
[libmtev eventer documentation](http://circonus-labs.github.io/libmtev/config/eventer.html).

### irondb-modules

IRONdb supports loading dynamically loadable modules that can provide optional features to an appliction.
Currently lua extensions are implemented as a module.
Others modules might be added in the future.
The module system is configured using the following stanza:

```
<include file="irondb-modules.conf" />
```

IRONdb modules are included from two files:

* `irondb-modules-stock.conf`, containing vendor controlled module configuration
* `irondb-modules-site.conf`, for site specific module configuration.

More details about the IRONdb module system can be found in
the [libmtev module documentation](http://circonus-labs.github.io/libmtev/config/modules.html).

### cache

```
<cache cpubuckets="128" size="0"/>
```

An LRU cache of open filehandles for numeric metric rollups. This can improve
rollup read latency by keeping the on-disk files for frequently-accessed
streams open.

#### cache cpubuckets

The cache is divided up into the specified number of "buckets" to facilitate
concurrent access by multiple threads. This parameter rarely requires tuning.

Default: 128

#### cache size

Size of the LRU cache.

Default: 0

Note that if you increase this value, you may need to raise your operating
system's resource limit on open files for the IRONdb process.

> This parameter is licensed as `cache_size` and may not be set higher than
> this value. If your license does not specify `cache_size` then you may not
> change this parameter.

### logs

Libmtev logging configuration. See the [libmtev logging
documentation](http://circonus-labs.github.io/libmtev/config/logging.html).

By default, the following log files are written and automatically rotated, with
the current file having the base name and rotated files having an
epoch-timestamp suffix denoting when they were created:
* `/irondb/logs/errorlog`: Output from the daemon process, including not just
  errors but also operational warnings and other information that may be useful
  to Circonus Support.
  * Rotated: 24 hours
  * Retained: 1 week
* `/irondb/logs/startuplog`: Additional non-error initialization output.
  * Rotated: 24 hours
  * Retained: 1 week
* `/irondb/logs/accesslog`: Logs from the REST API, including metric writes and
  reads as well as inter-node communication.
  * Rotated: 1 hour
  * Retained: 1 week

### listeners

Libmtev network listener configuration. See the [libmtev listener
documentation](http://circonus-labs.github.io/libmtev/config/listeners.html).

Each listener below is configured within a `<listener>` node. Additional
listeners may be configured if desired, or the specific address and/or port may
be modified to suit your environment.

#### Main listener

```
<listener address="*" port="8112" backlog="100" type="http_rest_api">
  <config>
    <document_root>/opt/circonus/share/snowth-web</document_root>
  </config>
</listener>
```

The main listener serves multiple functions:
* [HTTP REST API](api.md)
* [Cluster replication](operations.md#replication) (TCP) and gossip (UDP)
* [Operations Dashboard](operations.md#operations-dashboard)
* JSON-formatted node statistics (`http://thisnode:thisport/stats.json`)

##### Main listener address

The IP address on which to listen, or the special `*` to listen on any local IP
address.

Default: `*`

##### Main listener port

The port number to listen on. For the main listener this will utilize both TCP
and UDP.

Default: 8112

##### Main listener backlog

The size of the queue of pending connections. This is used as an argument to
the standard `listen(2)` system call. If a new connection arrives when this
queue is full, the client may receive an error such as ECONNREFUSED.

Default: 100

##### Main listener type

The type of libmtev listener this is. The main listener is configured to be
only a REST API listener. This value should not be changed.

Default: http_rest_api

#### Graphite listener

```
<listener address="*" port="2003" type="graphite">
  <config>
    <check_uuid>00000000-0000-0000-0000-000000000000</check_uuid>
    <check_name>mycheckname</check_name>
    <account_id>1</account_id>
  </config>
</listener>
```

The Graphite listener operates a Carbon-compatible submission pathway using the
[plaintext
format](http://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol).

Multiple Graphite listeners may be configured on unique ports and associated
with different check UUIDs. See the section on [Graphite
ingestion](graphite-ingestion.md) for details.

##### Graphite listener address

The IP address on which to listen, or the special `*` to listen on any local IP
address.

Default: `*`

##### Graphite listener port

The TCP port number to listen on.

Default: 2003

##### Graphite listener type

The type of listener. IRONdb implements a Graphite-compatible handler in
libmtev, using the custom type "graphite".

Default: graphite

##### Graphite listener config

These configuration items control which check UUID, name, and account ID are
associated with this listener. The first Graphite listener is configured during
[initial installation](installation.md).
* `check_uuid` is the identifier for all metrics ingested via this listener.
* `check_name` is a meaningful name that is used in
  [namespacing](graphite-ingestion.md#namespacing).
* `account_id` is also part of namespacing, for disambiguation.

#### Pickle listener

The Pickle listener operates a Carbon-compatible submission pathway using the
[pickle
format](http://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-pickle-protocol).

Its configuration is identical to the plaintext listener, except the type is `graphite_pickle`.


#### CLI listener

```
<listener address="127.0.0.1" port="32322" type="mtev_console">
  <config>
    <line_protocol>telnet</line_protocol>
  </config>
</listener>
```

The CLI listener provides a local [telnet
console](http://circonus-labs.github.io/libmtev/operations/telnet_console.html)
for interacting with libmtev subsystems, including modifying configuration. As
there is no authentication mechanism available for this listener, it is
recommended that it only be operated on the localhost interface.

##### CLI listener address

The IP address on which to listen, or the special `*` to listen on any local IP
address.

Default: 127.0.0.1

##### CLI listener port

The TCP port number to listen on.

Default: 32322

##### CLI listener type

The CLI listener uses the built-in libmtev type "mtev_console" to allow access
to the telnet console.

Default: mtev_console

### pools

```
<pools>
  <rollup concurrency="1"/>
  <nnt_put concurrency="16"/>
  <raw_writer concurrency="4"/>
  <raw_reader concurrency="16"/>
  <rest_graphite_numeric_get concurrency="4"/>
  <rest_graphite_find_metrics concurrency="4"/>
  <rest_graphite_fetch_metrics concurrency="10"/>
</pools>
```

Resource pools within IRONdb are used for various functions, such as reading
and writing metric data. Some aspects of pool behavior are configurable,
typically to adjust the number of worker threads to spawn.

The defaults presented are widely applicable to most workloads, but may be
adjusted to improve throughput. Use caution when raising these values too high,
as it could produce thrashing and _decrease_ performance.

If in doubt, [contact support](contact.md).

#### pools rollup concurrency

The number of unique metric names (UUID + metric name) to process in parallel
when performing rollups. A higher number generally causes the rollup operation
to finish more quickly, but has the potential to overwhelm the storage
subsystem if set too high.

Default: 1

> These tasks compete with other readers of the `raw_database`, so if `rollup`
> concurrency is set higher than 4x `raw_writer` concurrency, it cannot be
> reached.

#### pools nnt_put concurrency

The number of threads used for writing to numeric rollup files. Writes to a
given rollup file will always occur in the same queue.

Default: the number of physical CPU cores present during installation

#### pools raw_writer concurrency

The number of threads used for writing to the raw metrics database.
Additionally, by default, IRONdb will use 4x this number of threads for reading
from the raw metrics database.

Default: 4

#### pools raw_reader concurrency

The number of threads used for reading from the raw metrics database.

Default: (raw_writer concurrency * 4)


#### pools rest_graphite_numeric_get concurrency

The number of threads used for handling Graphite fetches. This is a general
queue for all fetch operations, and there are two other thread pools for
specific tasks within a fetch operation (see below.)

Default: 4

#### pools rest_graphite_find_metrics concurrency

The number of threads used for resolving metric names prior to fetch.

Default: 4

#### pools rest_graphite_fetch_metrics concurrency

The number of threads used for actually fetching Graphite metrics, including
those local to the node and those residing on remote nodes.

Default: 10

### rollups

```
<rollups>
  <rollup period="60" directory="/irondb/data/{node}/1m"/>
  <rollup period="300" directory="/irondb/data/{node}/5m"/>
  <rollup period="1800" directory="/irondb/data/{node}/30m"/>
  <rollup period="10800" directory="/irondb/data/{node}/3h"/>
</rollups>
```

Numeric rollups are produced from the raw metrics database. The schedule for
when to produce these rollups is controlled in the [raw database
configuration](#rawdatabase).

Each desired rollup is configured with a `period`, in seconds, and a named
`directory` for where to store that rollup's files. There will be one file per
unique metric name, per rollup period.

Configuring rollups involves tradeoffs between:
* Disk space
* Rollup execution time
* Granularity when fetching rolled-up data

In general, the `raw_database` will cover the most recent time period with very
good granularity. If you do not require this much resolution when viewing
historic data you can eliminate the finer-grained rollups from your
configuration.

### nntbs
```
<nntbs path="/irondb/nntbs/{node}">
  <shard period="60" size="1d" retention="1y" />
  <shard period="300" size="5d" retention="2y" />
  <shard period="1800" size="30d" retention="2y" />
  <shard period="10800" size="180d" retention="10y" />
</nntbs>
```

NNTBS is an optional more efficient rollup storage engine for data once it proceeds
past the [raw database](#rawdatabase).  If you don't have an `<nntbs>` stanza in your
config file, normal file based storage of NNT data will be used instead.

> All new installations will come with NNTBS on by default.

The `<shard>` options should match the periods defined in the [rollups section](#rollups).
For each `period` we are defining how much time each chunk of data should cover before creating
a new chunk. The minimum size for a shard is `127 * period`; for a 60 second period this would 
be `7620` seconds.  Whatever period you provide here will be rounded up to that multiple.  If
I provided `1d` as in the defaults above I would actually get `91440` seconds instead of `86400`.
The `retention` setting for each shard determines how long to keep this data on disk
before deleting it permanently.  `retention` is optional and if you don't provide it,
IRONdb will keep the data forever.  When a timeshard is completely past the `retention` limit
based on the current time, the entire shard is removed from disk.

> NOTE: for installations with high a cardinality of metric names you will want to reduce
> these `size` parameters to keep the shards small to ensure performance remains consistent.

Whatever settings are chosen here cannot be changed after the database starts writing data
into NNTBS (except for `retention`).  If you change your mind about sizing you will have to 
wipe and reconstitute each node in order to apply new settings.

If NNT files exist when NNTBS is activated, they will be converted to NNTBS
format the first time they are read, and the NNTBS data will be used to satisfy
the read request. Once the conversion is complete, the NNT file will be
deleted.

Alternatively, you may choose to do a full, offline conversion of NNT to NNTBS
by using the `-N` [command-line
option](/command-line-options.md#loader-options). In environments where a large
portion of the stored metrics are read frequently, the "lazy conversion" mode
described above may place too much load on the system when combined with normal
operations. You may wish to use this mode on one node at a time across your
cluster. It will not participate in the cluster while the conversion is
underway, and new incoming data will be journaled on other nodes until it
returns to normal service.

### raw_database

```
<raw_database location="/irondb/raw_db/{node}"
              data_db="nomdb"
              granularity="1d"
              min_delete_age="3d"
              delete_after_quiescent_age="12hr"
              max_clock_skew="1d"
              conflict_resolver="abs_biggest"
              rollup_strategy="raw_iterator"
/>
```

Raw metrics database. This stores all ingested metrics at full resolution for a
configurable period of time, after which the values are rolled up and stored in
one or more [period-specific files](#rollups).

Time periods are specified as second-resolution [libmtev
durations](http://circonus-labs.github.io/libmtev/apireference/c.html#mtevgetdurationss).

The `location` and `data_db` attributes should not be modified.

#### raw_database granularity

Granularity controls the sharding of the raw database. A shard is the unit of
data that will be rolled up and removed, after a configurable age and period of
quiescence (no new writes coming in for that shard.)

> Do not change granularity after starting to collect data, as this will result
> in data loss.

Default: 1 day

#### raw_database min_delete_age

The minimum age that a shard must be before it is considered for rollup and
deletion.

Default: 3 days

#### raw_database delete_after_quiescent_age

The period after which a shard, if it has been rolled up and not subsequenty
written to, may be deleted.

Default: 12 hours

#### raw_database max_clock_skew

Allow the submission of metrics timestamped up to this amount of time in the
future, to accommodate clients with incorrect clocks.

Default: 1 day

#### raw_database conflict_resolver

When a metric gets written more than one time at the exact millisecond
offset you have a conflict we have to resolve. All operations in IRONdb are
commutative and this lets us avoid complicated consensus algorithms for data.
Conflicts, therefore, need to choose a winner and this choice needs to be
consistent across the cluster. IRONdb gives you the following choices for
conflict resolution should a datapoint appear more than once at the same
millisecond.

* `abs_biggest` - save the largest by absolute value.
* `last_abs_biggest` - if used with the [IRONdb-relay](irondb-relay.md)
  aggregation capabilities the datapoints can track a generation counter. This
  resolver considers the generation of the datapoint and then uses the largest
  by absolute value if the generations collide. If you are not using the relay,
  this will fall back to the same behavior as `abs_biggest`.
* `abs_smallest` - save the smallest by absolute value.
* `last_abs_smallest` - same as `last_abs_biggest` but smallest instead.
* `last_biggest` - same as `last_abs_biggest` but uses the largest without
  absolute value.
* `last_smallest` - same as last but smallest.
* `biggest` - the larger value without absolute.
* `smallest` - the smaller value without absolute.

This setting should be the same on all nodes of the IRONdb cluster.

Default: "abs_biggest"

#### raw_database rollup_strategy

Control how rollups are performed.  By default, all levels of rollup data are
calculated from the raw database as it is iterated.

Prior to version `0.12` the default if not specified was that the lowest level
of rollup was computed and then IRONdb would read this lowest level data and
compute higher level rollups. This rollup strategy has been removed.

Default: "raw\_iterator"


### metric_name_database

> This database is no longer used as of version `0.12`.

```
<metric_name_database location="/irondb/metric_name_db/{node}"
              query_cache_size="1000"
	      query_cache_timeout="900"
/>
```

The database of stored metric names.  This database is used to satisfy graphite /metrics/find
queries.  By default, this database will cache 1000 queries for 900 seconds.  Any newly
arriving metric names will invalidate the cache so subsequent queries are correct.

#### metric_name_database location

The location on disk where the database files reside

#### metric_name_database query_cache_size

The number of incoming /metrics/find queries to cache the results for.

#### metric_name_database query_cache_timeout

The number of seconds that cached queries should remain in the cache before being 
expired.

### journal

```
<journal concurrency="4"
         max_bundled_messages="25000"
         pre_commit_size="131072"
         send_compressed="true"
         use_indexer="false"
/>
```

Journals are write-ahead logs for replicating metric data to other nodes. Each
node has one journal for each of its cluster peers.

#### journal concurrency

Establishes this number of concurrent threads for writing to each peer journal,
improving ingestion throughput.

Default: 4

> A concurrency of 4 is enough to provide up to 700K measurements/second
> throughput, and is not likely to require adjustment except in the most
> extreme cases.

#### journal max_bundled_messages

Outbound journal messages will be sent in batches of up to this number,
improving replication speed.

Default: 25000

#### journal pre_commit_size

An in-memory buffer of this number of bytes will be used to hold new journal
writes, which will be flushed to the journal when full. This can improve
ingestion throughput, at the risk of losing up to this amount of data if the
system should fail before commit. To disable the pre-commit buffer, set this
attribute to 0.

Default: 131072 (128 KB)

#### journal send_compressed

When sending journal messages to a peer, compress the messages before sending
to save bandwidth, at the cost of sligtly more CPU usage. The bandwidth savings
usually outweight the cost of compression.

Default: true

#### journal use_indexer

Spawn a dedicated read-ahead thread to build
[JLog](https://github.com/omniti-labs/jlog) indexes of upcoming segments in the
write-ahead log for each remote node. This is only needed in the most extreme
cases where the highest replication throughput is required. Almost all other
installations will not notice any slowdown from indexing "on demand", as new
segments are encountered.

Note that this will spawn one extra thread per journal (there is one journal
for every remote node in the cluster.) For example, activating this feature
will spawn 15 additional threads on each node in a 16-node cluster.

Default: false

### topology

```
<topology path="/opt/circonus/etc/irondb-topo"
          active="(hash value)"
          next=""
          redo="/irondb/redo/{node}"
/>
```

The topology node instructs IRONdb where to find its current cluster
configuration. The `path` is the directory where the imported topology config
lives, which was created during setup. `active` indicates the hash of the
currently-active topology. `next` is currently unused.

No manual configuration of these settings is necessary.

## circonus-watchdog.conf

### watchdog

```
<watchdog glider="/opt/circonus/bin/backwash" tracedir="/opt/circonus/traces"/>
```

The watchdog configuration specifies a handler, known as a "glider", that is to
be invoked when a child process crashes or hangs. See the [libmtev watchdog
documentation](http://circonus-labs.github.io/libmtev/config/watchdog.html).

If [crash handling](operations.md#crash-handling) is turned on, the `glider` is
what invokes the tracing, producing one or more files in the `tracedir`.
Otherwise, it just reports the error and exits.

## licenses.conf

This file holds any and all licenses that apply to this IRONdb node. Refer to
the [installation steps](installation.md#add-license) for details on obtaining
and installing licenses.

The IRONdb license governs the following functionality:

* License term (`<expiry>`)

After this unix timestamp the license is invalid and will no longer
work for any of the below.

* Ingest cardinality (`<max_streams>`)

How many unique time series (uniquely named streams of data) this installation
can ingest in the most recent 5 minute period.

This number applies to all nodes in the cluster although each node applies
this restriction individually.  The math for unique streams is an estimate
in the past 5 minutes and you are given a 15% overage before ingestion is affected.

If this license is violated, ingestion will stop for the remainder of the 5 minute
period that the violation was detected.  After the 5 minute period ends, the counter
will reset to test the new 5 minute period.

* NNT cache size (`<cache_size>`)

This governs how large of a memory cache (max) is allowed for caching open NNT file 
handles.  This has no impact on NNTBS configured systems.

Deprecated if you have upgraded to NNTBS.

* Enabledment of Lua extensions (`<lua_extension>`)

Whether or not Lua extensions will operate.

* Stream tags support (`<stream_tags>`)

Whether or not stream tag related API calls and stream tag ingestion will work.
If you do not have this license and stream tagged data arrives it will be silently
discarded.

* Histogram support (`<histograms>`)

Whether or not histograms can be ingested.  If you do not have this license and 
attempt to ingest histogram data it will be silently discarded.

* Text metric support (`<text>`)

Whether or not text metrics can be ingested.  If you do not have this license and 
attempt to ingest text data it will be silently discarded.


If you are interested in any of the above functionality and do not currently have a license
please contact [sales@circonus.com](mailto:sales@circonus.com) to upgrade your license.

