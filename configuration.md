# IRONdb Configuration Options

IRONdb is implemented using
[libmtev](https://github.com/circonus-labs/libmtev/), a framework for building
high-performance C applications. You may wish to review the libmtev
[configuration documentation](http://circonus-labs.github.io/libmtev/config/)
for an overview of how libmtev applications are configured generally.

This document deals with options that are specific to IRONdb, but links to
relevant libmtev documentation where appropriate.

## irondb.conf

This is the primary configuration file that IRONdb reads at start. It includes
additional configuration files which are discussed later.

### snowth

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

> Text-type metrics are not currently supported in IRONdb, since Graphite has
> no way to represent them.

### eventer

Libmtev eventer system configuration. See the [libmtev eventer
documentation](http://circonus-labs.github.io/libmtev/config/eventer.html).

### cache

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

#### Graphite listener

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

### pools

```
<pools>
  <nnt_put concurrency="16"/>
  <raw_writer concurrency="4"/>
</pools>
```

Resource pools within IRONdb are used for various functions, such as reading
and writing metric data. Some aspects of pool behavior are configurable,
typically to adjust the amount of concurrency within a given pool.

The defaults presented are widely applicable to most workloads, but may be
adjusted to improve throughput. Use caution when raising these values too high,
as it could produce thrashing and _decrease_ performance.

#### pools nnt_put concurrency

The number of job queues used for writing to numeric rollup files. Writes to a
given rollup file will always occur in the same queue.

Default: the number of physical CPU cores present during installation

#### pools raw_writer concurrency

The number of job queues used for writing to the raw metrics database.
Additionally, IRONdb will use 4x this number of job queues for reading from the
raw metrics database.

Default: 4

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

### raw_database

```
<raw_database location="/irondb/raw_db/{node}"
              data_db="nomdb"
              granularity="1d"
              min_delete_age="3d"
              delete_after_quiescent_age="12hr"
              max_clock_skew="1d"
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

> Do not change granularity after starting to collect data, as this may result
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

The topology node instructs IRONdb where to find its current cluster
configuration. The `path` is the directory where the imported topology config
lives, which was created during setup. `active` indicates the hash of the
currently-active topology. `next` is currently unused.

No manual configuration of these settings is necessary.

## circonus-watchdog.conf

### watchdog

The watchdog configuration specifies a handler that is to be invoked when a
child process crashes or hangs. See the [libmtev watchdog
documentation](http://circonus-labs.github.io/libmtev/config/watchdog.html).

If [crash handling](operations.md#crash-handling) is turned on, the glider is
what invokes the tracing. Otherwise, it just reports the error and exits.

## licenses.conf

This file holds any and all licenses that apply to this IRONdb node. Refer to
the [installation steps](installation.md#add-license) for details on obtaining
and installing licenses.
