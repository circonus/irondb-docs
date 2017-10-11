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
<eventer>
  <config>
    <ssl_dhparam1024_file/>
    <ssl_dhparam2048_file/>
  </config>
</eventer>
```

Libmtev eventer system configuration. See the [libmtev eventer
documentation](http://circonus-labs.github.io/libmtev/config/eventer.html).

The `ssl_dhparam*_file` configurations are set to null to disable automatic
Diffie-Hellman parameter generation at startup. IRONdb does not utilize TLS by
default, though this capability is present in libmtev.

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
Additionally, IRONdb will use 4x this number of threads for reading from the
raw metrics database.

Default: 4

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

### raw_database

```
<raw_database location="/irondb/raw_db/{node}"
              data_db="nomdb"
              granularity="1d"
              min_delete_age="3d"
              delete_after_quiescent_age="12hr"
              max_clock_skew="1d"
              conflict_resolver="abs_biggest"
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

### journal

```
<journal concurrency="4"
         max_bundled_messages="25000"
         pre_commit_size="131072"
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
