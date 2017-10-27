## IRONdb-relay

The IRONdb-relay, like the carbon-relay or the carbon-c-relay is a metrics data
router that takes carbon TEXT format metrics and routes them to the appropriate
IRONdb storage node. 

Since IRONdb uses SHA256 hashing to route metrics to IRONdb nodes, it is
incompatible with routing options that exist in carbon-c-relay and carbon-relay.
In addition, it provides advanced aggregation and filtering functions.

Features
===========

* Ingests TEXT carbon format metrics on a configurable port

  `foo.bar.baz 1234.56 1507724786`
  
* Routes to primary owner of the metric name and then subsequent nodes if the
  primary is down.

* [Aggregation](#modules) of incoming metrics based on regular expressions with support for
  SUM, AVG, MIN, MAX, p0, p25, p50, p95, p99, p100
  
* [Blacklist and whitelist filtering](#modules) of metrics based on regular expressions
  
* Durable delivery of metrics using write head logs
  
Installation
================

IRONdb-relay requires one of the following operating systems:
* RHEL/CentOS, version 6.x, 7.x.
* Ubuntu 16.04 LTS.

The following network protocols and ports are utilized. These are defaults and
may be changed via configuration files.

* 2003/tcp (Carbon plaintext submission)
* 8112/tcp (admin UI, HTTP REST API)

### System Tuning

You should follow the same system tuning as outline in
the [IRONdb installation](installation.md#system-tuning).

### Configure Software Sources

Use the same software source as
the [IRONdb installation](installation.md#configure-software-sources) for EL7 or
Ubuntu.

For EL6, use the same config as EL7 but replace `7` with `6` in the `baseurl`
for `[circonus]` and replace `el7` with `el6` in the `baseurl` for
`[circonus-crash-reporting]`

### Install Package

* (EL6, EL7) `/usr/bin/yum install circonus-platform-irondb-relay`

* (Ubuntu) `/usr/bin/apt-get install circonus-platform-irondb-relay`


### Run Installer

Prepare site-specific information for setup. These values may be set via shell
environment variables, or as arguments to the setup script. The environment
variables are listed below.
   * ##### IRONDB\_CHECK\_UUID

     *\(required\)* Check ID for Graphite metric ingestion, which must be the
     same on all cluster nodes. You may use the `uuidgen` command that comes
     with your OS, or generate a UUID with an external tool or website.
   * ##### IRONDB\_CHECK\_NAME

     *\(required\)* The string that will identify Graphite-compatible metrics
     stored in the check identified by `IRONDB_CHECK_UUID`. For example, if you
     submit a metric named "my.metric.1", and the check is named "test", the
     resulting metric name in IRONdb will be "graphite.test.my.metric.1".
   * ##### IRONDB\_BOOTSTRAP

     *\(required\)* The comma separated list of IRONdb nodes (ipaddress:port) to
     use to discover the layout of the IRONdb cluster. It's a good practice to
     list all IRONdb nodes in this list to adjust to down nodes.
   * ##### IRONDB\_CRASH\_REPORTING

     *\(optional\)* Control enablement of automated crash reporting. Default is
     "on". IRONdb utilizes sophisticated crash tracing technology to help
     diagnose errors. Enabling crash reporting requires that the system be able
     to connect out to the Circonus reporting endpoint:
     https://circonus.sp.backtrace.io:6098 . If your site's network policy
     forbids this type of outbound connectivity, set the value to "off".
   * ##### IRONDB\_RELAY\_DURABLE

     *\(optional\)* Control enablement of durable delivery.  Default is "false".
     If set to "true", will cause IRONdb-relay to use the disk to persist all 
     incoming metrics to the file system before sending them on to IRONdb nodes.

Run the setup script. All required options must be present, either as
environment variables or via command-line arguments. A mix of environment
variables and arguments is permitted, but environment variables take precedence
over command-line arguments. Use the `-h` option to view a usage summary:

    Usage: setup-irondb-relay [-h] -c <check-name> -u <check-uuid> -B <addr:port>[,<addr:port>,...]
           [-d] [-b (on|off)]
      -c <check-name>       : Graphite check name
      -u <check-uuid>       : Graphite check UUID
      -d                    : Use durable delivery to IRONdb
      -B <irondb-node-list> : Bootstrap to this list of IRONdb nodes
      -b on|off             : Enable/disable crash reporting (default: on)
      -h                    : Show usage summary

    Example:
      setup-irondb-relay -c foo -u f2eaa1b7-f7e8-41bd-9e8d-e52d43dc88b0 -d -B 10.1.13.1:8112,10.1.13.2:8112 -b on

The setup script will configure your IRONdb-relay instance and start the
service. See the [Graphite Ingestion](./graphite-ingestion.md) section for
details.


Configuration
==================

IRONdb-relay is implemented using
[libmtev](https://github.com/circonus-labs/libmtev/), a framework for building
high-performance C applications. You may wish to review the libmtev
[configuration documentation](http://circonus-labs.github.io/libmtev/config/)
for an overview of how libmtev applications are configured generally.

This document deals with options that are specific to IRONdb-relay, but links to
relevant libmtev documentation where appropriate.

Default values are those that are present in the default configuration produced
during initial installation.

## irondb-relay.conf

This is the primary configuration file that IRONdb-relay reads at start. It includes
additional configuration files which are discussed later.  It is located at
`/opt/circonus/etc/irondb-relay.conf`

### irondb-relay

```
<irondb-relay lockfile="/irondb-relay/logs/irondb-relay.lock" text_size_limit="512">
```

IRONdb-relay's libmtev application name. This is a required node and **must not** be
changed.

#### irondb-relay lockfile

Path to a file that prevents multiple instances of the application from running
concurrently. You should not need to change this.

Default: `/irondb-relay/logs/irondb-relay.lock`

### eventer

```
<eventer>
  <config>
    <concurrency>8</concurrency>
    <default_queue_threads>8</default_queue_threads>
    <ssl_dhparam1024_file/>
    <ssl_dhparam2048_file/>
  </config>
</eventer>
```

Libmtev eventer system configuration. See the [libmtev eventer
documentation](http://circonus-labs.github.io/libmtev/config/eventer.html).

The `ssl_dhparam*_file` configurations are set to null to disable automatic
Diffie-Hellman parameter generation at startup. IRONdb-relay does not utilize TLS by
default, though this capability is present in libmtev.

### logs

Libmtev logging configuration. See the [libmtev logging
documentation](http://circonus-labs.github.io/libmtev/config/logging.html).

By default, the following log files are written and automatically rotated, with
the current file having the base name and rotated files having an
epoch-timestamp suffix denoting when they were created:
* `/irondb-relay/logs/errorlog`: Output from the daemon process, including not just
  errors but also operational warnings and other information that may be useful
  to Circonus Support.
  * Rotated: 24 hours
  * Retained: 1 week
  
### modules

Libmtev module configuration. See
the
[libmtev module documentation](http://circonus-labs.github.io/libmtev/config/modules.html)

There are 2 modules provided with IRONdb-relay:

* filter

  Will allow you to setup whitelist/blacklist filtering for metrics
  
  1. Enable the module under the `<modules>` section of your config by adding the line:
  
     `<generic image="filter" name="filter_hook"></generic>`
  
  2. Create your filter config
  
     Add a `<filter>` block to your `irondb-relay.conf` file. A `<filter>` can
     have exactly one `<ruleset>` block. A `<ruleset>` block can have any
     number of `<rule>` blocks. A `<rule>` block consists of a `<match_regex>`
     or `<match_all>` directive and a `<result>`. `<rule>` blocks are processed
     in order and processing stops at the first matching `<rule>`.
     
     Depending on whether you want a whitelist or a blacklist you would either
     configure your filter to whitelist a set of regexes and then have a
     `<match_all>` rule to `deny` everything else, or you would configure your
     filter to have a rule to match metrics you want to blacklist then have a
     final `<match_all>` rule to `allow` the remainder.
     
     An example of a blacklist would resemble:
     
     ``` xml 
      <filter> 
       <ruleset> 
         <rule> 
           <match_regex>^relay_test\.agent\.2.*</match_regex>
           <result>deny</result> 
         </rule> 
         <rule> 
           <match_all>true</match_all>
           <result>allow</result> 
         </rule> 
      </ruleset>
     </filter>
     ```
     
     The above would blacklist everything that starts `relay_test.agent.2` and
     allow everything else.
     
     For best performance, it is wise to organize your `<rule>` blocks in
     descending order based on the expected frequency of matching. You want the
     `<rule>`s that match more often to be at the beginning of the list and the
     `<rule>`s that match infrequently to be lower down in the list.
    
* aggregation_hook

  Will allow you to perform aggregation on incoming metrics and produce new
  metrics as the result.

  1. Enable the module under the `<modules>` section of your `irondb-relay.conf` by adding the line:

    `<generic image="aggregation_hook" name="aggregation_hook"></generic>`
    
  2. Create your aggregation config
  
    Add an `<aggregation>` block to your `irondb-relay.conf` file.  An `<aggregation>` can
    have exactly one `<matchers>` block which itself can contain any number of `<matcher>` blocks.
    A `<matcher>` block consists of the following:
    
    * `<match_regex>` - the regular expression (including captures) you want to match incoming metrics
    * `<flush_seconds>` - how long to aggregate matching records for 
    * `<flush_name_template>` - the template to use for the outgoing name of the aggregated metric result 
    * `<flush_functions>` - a comma separate list of functions you want applied to the matching metric values
    * `<flush_original>` - whether or not you want to let the original incoming metric to also be sent to IRONdb 
    * `<jitter_ms>` - to prevent collisions among multiple relays which might be aggregating the same metrics, set this to a unique value per irondb-relay instance
    * `<idle_cycles>` - how many multiples of `<flush_seconds>` should the relay wait before giving up on any new incoming metrics that would fall into this aggregation window

    For `<flush_name_template>` you can use capture references (`\1`) and a special sequence `${FF}` to create the outgoing metric name.

    An example:
    
    ``` xml 
    <aggregation>
    <matchers>
      <matcher>
        <match_regex>^relay_test\.agent.[0-9]*\.metrics\.([0-9]*)</match_regex>
        <flush_seconds>10</flush_seconds>
        <flush_name_template>agg.all_agents.metrics.\1_${FF}</flush_name_template>
        <flush_functions>avg</flush_functions>
        <flush_original>false</flush_original>
        <jitter_ms>10</jitter_ms>
        <idle_cycles>2</idle_cycles>
      </matcher>
      <matcher>
        <match_regex>^relay_test\.agent.[0-9]*\.metrics\.([0-9]*)</match_regex>
        <flush_seconds>10</flush_seconds>
        <flush_name_template>foo.all_agents.metrics.\1_${FF}</flush_name_template>
        <flush_functions>sum</flush_functions>
        <flush_original>false</flush_original>
        <jitter_ms>10</jitter_ms>
        <idle_cycles>2</idle_cycles>
      </matcher>
    </matchers>
    </aggregation>
    ```
    
    The above first `<matcher>` matches incoming metrics that start
    `relay_test.agent.`, followed by any number of digits, followed by
    `.metrics.` and finally capturing the trailing sequence of any number of
    digits. An example metric that would match: `relay_test.agent.5.metrics.27`.
    All incoming rows that fit this regex will have their values averaged
    (`<flush_functions>avg</flush_functions>`) for 10 seconds. The original
    source rows will *not* be sent to IRONdb and after 10 seconds a new row will
    be produced that looks like: `agg.all_agents_metrics.27_avg` and it's value
    will be the average of `metrics.27` from all agents that this relay saw in
    that 10 second window.
    
    The 2nd `<matcher>` performs the same match but uses `sum` instead of `avg` and uses a 
    different `<flush_name_template>`.
    
    The supported `<flush_functions>` are: `sum,avg,min,max,p0,p25,p50,p95,p99,p100,histogram`
    
    * `sum` is the sum of values of the matching rows.
    * `avg` is the mean 
    * `min` is the smallest value
    * `max` is the largest
    * `p0` is a synonym for `min`
    * `p100` is a synonym for `max`
    * `p25` is the 25th percentile value 
    * `p50` is the 50th percentile value
    * `p95` is the 95th percentile value 
    * `p99` is the 99th percentile value
    * `histogram` is the complete distribution of all values
    
    With `histogram` IRONdb will be able to store the histogram data but there
    currently is no facility in graphite-web to render this data.
    
    #### A note on flushing results to IRONdb
    
    >The very first row that matches and creates an aggregation "window" will
    >start the flush timer. `<flush_seconds>` later the result will be sent to
    >IRONdb. It is possible after this initial flush that some late data arrives
    >that would normally fit into that same aggregation window. This is where
    >`<idle_cycles>` comes into play. The relay will retain the aggregation
    >window until no more matching rows are seen for `<idle_cycles>` cycles. If
    >matching rows *are* seen the aggregation window is updated with the new
    >values and the record is re-sent to IRONdb. To control the behavior of
    >conflicts within the database when this happens please see
    >the [conflict resolver](configuration.md#rawdatabase-conflictresolver) section of
    >the IRONdb configuration.
    
### send

This config has a single attribute: `durable="true|false"`. If set to "true" it
will use the `<journal>` settings below to journal every row destined for IRONdb
nodes. If set to "false", it will bypass the journaling and directly send to
IRONdb. If set to "false", the relay will do its best to make sure data arrives
at one of the IRONdb nodes if the primary doesn't respond or is down but there
is no guarantee of delivery.

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
* [Operations Dashboard](#operations-dashboard)
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
with different check UUIDs. See the section
on [Graphite ingestion](graphite-ingestion.md) for details. The graphite
listener config here should be kept in sync with
the [same configuration](configuration.md#graphite-listener) for the IRONdb
nodes themselves.

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


### journal

```
<journal concurrency="4"
         max_bundled_messages="25000"
         pre_commit_size="131072"
/>
```

Journals are write-ahead logs for replicating metric data to IRONdb nodes. Each
IRONdb-relay has one journal for each of the IRONdb nodes.

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

## circonus-watchdog.conf

### watchdog

```
<watchdog glider="/opt/circonus/bin/backwash" tracedir="/opt/circonus/traces-relay"/>
```

The watchdog configuration specifies a handler, known as a "glider", that is to
be invoked when a child process crashes or hangs. See the [libmtev watchdog
documentation](http://circonus-labs.github.io/libmtev/config/watchdog.html).

If [crash handling](operations.md#crash-handling) is turned on, the `glider` is
what invokes the tracing, producing one or more files in the `tracedir`.
Otherwise, it just reports the error and exits.

Operations Dashboard
============================

IRONdb-relay comes with a built-in operational dashboard accessible via port
8112 (default) in your browser, e.g., http://irondb-relay-host:8112. This
interface provides real-time information about the IRONdb-relay. There are a
number of tabs in the UI, which display different aspects about the node's
current status.

The node's version info is displayed at top right.

### Overview Tab

The "Overview" tab displays top level statistics about the relay.

#### Inflow

* Socket accepts - how many connections have been made to this relay since startup
* Received - how many individual rows have been sent into this relay
* Parsed - the number of rows that we successfully parsed and validated
* Parse errors - the number of parse failures
* Aggregation gated - the number of rows that were *not* send on to IRONdb
  because of [filtering](#modules) or [aggregation](#modules)

#### Outflow

* Rows sent - the number of rows sent to IRONdb nodes
* Batches sent - rows are sent in batches, this is the count
* Batches OK - successful batch count
* Batch timeouts - the count of batches that timed out while sending to IRONdb nodes


### Durable Delivery Tab

If `<send durable="true" />` is set in the [config](#send), this tab will
contain information about replication lag.

Each IRONdb node will be listed along with the number of journal reads and
writes and how far behind this relay is in sending to each IRONdb node. Ideally
we should have `Seconds behind` under 10 seconds.

### Filters Tab

If you have the filter module enabled, lists each filter in your current `<ruleset>` and how many rows it has processed.

### Aggregation Tab

If you have the aggregation_hook module enabled, lists each aggregation and how
many rows it has seen, matched, skipped, and generated.

### Internals Tab

Shows internal application information, such as recent error logging, job
queues, open sockets, and timers. This data is used by Circonus Support when
troubleshooting issues.
