# Operations

By default, IRONdb listens externally on TCP port 2003, TCP and UDP port 8112,
and locally on TCP port 32322. These ports can be changed via configuration
files. There are normally two processes, a parent and child. The parent process
monitors the child, restarting it if it crashes. The child process provides the
actual services, and is responsible for periodically "heartbeating" to the
parent to show that it is making progress.

IRONdb is sensitive to CPU and IO limits. If either resource is limited, you
may see a process being killed off when it does not heartbeat on time. These
are known as "watchdog" events.

## Service Management

The IRONdb service is called `circonus-irondb`.

To view service status: `/bin/systemctl status circonus-irondb`

To start the service: `/bin/systemctl start circonus-irondb`

To stop the service: `/bin/systemctl stop circonus-irondb`

To restart the service: `/bin/systemctl restart circonus-irondb`

To disable the service from running at system boot: `/bin/systemctl disable circonus-irondb`

To enable the service to run at system boot: `/bin/systemctl enable circonus-irondb`

## Logs

Log files are located under `/irondb/logs` and include the following files:
 * accesslog
 * errorlog

The access logs are useful to verify activity going to the server in question. Error logs record, among other things, crashes and other errant behavior, and may contain debugging information important for support personnel. The logs are automatically rotated and retained based on configuration attributes in `/opt/circonus/etc/irondb.conf`.

If the child process becomes unstable, verify that the host is not starved for
resources (CPU, IO, memory). Hardware disk errors can also impact IRONdb's
performance.  Install the `smartmontools` package and run `/usr/sbin/smartctl
-a /dev/sdX`, looking for errors and/or reallocated-sector counts.

## Crash Handling

Application crashes are, by default, automatically reported to Circonus, using [Backtrace.io](https://backtrace.io/) technology. When the crash occurs, a tracer program quickly gathers a wealth of detailed information about the crashed process and sends a report to Circonus, in lieu of obtaining a full core dump. 

If you have disabled crash reporting in your environment, you can still enable traditional core dumping.

On EL7:
* Add the following to `/opt/circonus/etc/irondb-node-config`: `ulimit -c unlimited`
* Restart the `circonus-irondb` service
* Set the kernel core pattern to place core dumps in a suitable location. See the core(5) man page for details. This location must be writable by the user that IRONdb runs as (`nobody`).
* Allow setuid dumps: `sysctl -w fs.suid_dumpable=1`

When a process crashes, a core dump will be created in `/irondb/appcrash` with the filename `core.<executable-name>.<pid>`, and the event will be recorded in the system log.

(TODO: Ubuntu)

## Debugging Mode

If instability continues, you may run IRONdb as a single process in the
foreground, with additional debugging enabled.

First, ensure the service is disabled: `/usr/bin/systemctl stop circonus-irondb`

Then, run the following as root:

    /opt/circonus/bin/irondb-start -D -d

Running IRONdb in the foreground with debugging should make the error apparent, and Circonus Support can help diagnose your problem. Core dumps are also useful in these situations (see above).

## Replication

In a multi-node cluster, IRONdb nodes communicate with one another using port 8112. Metric data are replicated over TCP, while intra-cluster state (a.k.a. [gossip](api/gossip-json.md)) is exchanged over UDP. The replication factor is determine by the number of [write copies](installation.md#determine-write-copies) defined in the cluster's toplogy. When a node receives a new metric data point, it calculates which nodes should "own" this particular stream, and, if necessary,  writes out the data to a local, per-node journal. This journal is then read behind and replayed to the destination node.

When a remote node is unavailable, its corresponding journal on the remaining active nodes continues to collect new metric data that is being ingested by the cluster. When that node comes back online, its peers begin feeding it their backlog of journal data, in addition to any new ingestion which is coming directly to the returned node. 

## Proxying

Clients requesting metric data from IRONdb need not know the specific location of a particular stream's data in order to fetch it. Instead, they may request it from any node, and if the data are not present on that node, the request is transparently proxied to a node that does have the data. Because nodes can fail and need to catch up with their peers, proxying favors remote nodes that are the most up to date. This is determined from the gossip data, which includes a latency metric, indicating the most recent replication message that this node has seen from each of its peers. The node performing the proxying decides which of the other nodes that own the given metric has the most recent data.

If gossip state is unavailable, such as due to a network partition, the node handling the request may return less recent data, if it proxies to a node that happens to be behind, or none at all, if the requested data is not available locally and all other owning nodes are unavailable.

## Operations Dashboard

IRONdb comes with a built-in operational dashboard accessible via port 8112 in your browser, e.g., http://irondb-host:8112. This interface provides real-time information about the IRONdb cluster. There are a number of tabs in the UI, which display different aspects about the node's current status.

The node's address and port are displayed at top right, along with a version hash.

### Overview Tab

The "Overview" tab displays the current throughput, available rollup dimensions, and storage statistics.

#### Ingestion

Read (Get) and Write (Put) throughput, per second.
 * "Batch" is an operation that reads or writes one or more metric streams.
 * "Tuple" is an individual measurement.

Therefore, a write operation that PUTs data for 10 different streams in a single operation counts as 1 Batch and 10 Tuples.

#### License

Displays details of the node's license.

#### Numeric Rollups

Displays throughput for both reads and writes per second for numeric rollup data.
 * "Cache Size" is the number of open file handles for numeric rollup data. A given stream's data may be stored in multiple files, one for each configured rollup period in which that stream's data has been recorded.
 * "Rollups" is the list of available rollup periods.

#### Histogram Rollups

Displays throughput for both reads and writes per second for histogram rollup data.
 * "Rollups" is the list of available rollup periods.

#### Text Changesets

Displays throughput for both reads and writes per second for text data.

#### Storage

Disk space used and performance data per data type and rollup dimension.

Each icon under "Performance" displays a histogram of the associated operation (Get/Put/Proxy) latency since the server last started. Latencies are plotted on the x-axis as seconds, with counts of operations in each latency bucket on the y-axis. The average latency for the set is displayed as a vertical green line.

Hovering over the x-axis will display a shaded region representing quantile bands and the latency values that fall within them. The quantiles are divided into four bands: p(0)-p(25), p(25)-p(50), p(50)-p(75), and p(75)-p(100). To avoid losing detail, the maximum x-axis values are not displayed, but the highest latency value may be seen by hovering over the p(75)-p(100) quantile band.

Hovering over an individual latency bar will display three lines at the top right corner of the histogram. These represent the number of operations that had less than, equal to, or greater than the current latency, and what percentage of the total each count represents.

The Used, Total, and Compress Ratio figures represent how much disk space is occupied by each data type or rollup, the total filesystem space available on the node, and the ratio of the original size to the compressed size stored on disk. The compression ratio is determined from the underlying ZFS filesystem.

### Replication Latency Tab

Displays the difference between the current time on each node and the time of the most recently received journal message from all other nodes. Each node's section may thus be read as "how far behind" that node is from its peers.

The node being viewed will be displayed with a blue bar, and the color of remote nodes will be green, yellow, or red, based on the difference between the current node's time and the timestamp of the last gossip message received from that node. All nodes should be running NTP or a similar time synchronization daemon. For example, if a remote node is shown as "(0.55 seconds old)", that means that a gossip message was received from that node 0.55 seconds ago, relative to the current node.

If the current node has never received a gossip message from a remote node since starting, that node will be displayed with a black bar, and the latency values will be reported as "unknown". This indicates that the remote node is either down or there is a network problem preventing communication with that node. Check that port 8112/udp is permitted between all cluster nodes.

### Topology Tab

Displays the layout of the topology ring, and how the key space is divided up among the participating nodes. A given stream may be located by entering its UUID and Metric Name in the fields provided, and then clicking the Locate button. Icons indicating the primary, secondary, and tertiary owners of the metric (or more if more write copies are configured) will appear next to the corresponding node.

### Extensions Tab

Displays a list of the loaded Lua extensions that provide many of the features of IRONdb.

### Internals Tab

Shows internal application information, such as job queues, open sockets, and timers. This data is used by Circonus Support when troubleshooting issues.
