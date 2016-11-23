# Operations

## Data Storage

The data storage service runs the snowth:default process, listening externally on port 8112 and locally on port 32322. Like the broker, this service has two processes: a child and a parent, which serves as watchdog for the child.

The data storage system requires ZFS, and therefore it is the only service to require the use of OmniOS to function. Basic OmniOS information and mailing lists can be found here: http://omnios.omniti.com/wiki.php/WikiStart. For further help, please contact Circonus Support (support@circonus.com).

Snowthd is sensitive to CPU and IO limits. If either resource is limited, you can see Snowthd child processes being killed off by the parents when they do not heartbeat on time.

Log files are located under /snowth/logs and include the following files:

 * acceslog

 * errorlog

The access logs are useful to verify activity going to the server in question. Error logs may contain debugging information for Support personnel.

If the snowthd child process becomes unstable, verify that the host is not starved for resources (CPU, IO, memory). Hardware disk errors can also impact snowth's performance. To check for errors run:

```
iostat -zxne
```

You will see an "Errors" section to the right. If you begin to see hardware errors there, this could indicate a disk failure. If in doubt, contact Circonus Support (support@circonus.com) for assistance.

If instability continues, you can run snowth in the foreground. First, determine the node's ID by doing an:

```
ls /snowth/data/
```

Then, run the following as root:

```
/opt/circonus/sbin/snowthd -D -d \

 -u nobody \

 -g nobody \

 -c /opt/circonus/etc/snowth.conf \

 -i <nodeid from previous command>
```

Like the broker, running snowth in the foreground should allow you to capture a core dump, which Circonus Support can use to diagnose your problem.

### Operations Dashboard

Snowth comes with built-in operational dashboard accessible from any data storage host on port 8112 in your browser, e.g., http://snowthhost:8112. This interface provides real-time information about the data storage cluster.

The "Overview" tab tells you the throughputs of data gets and puts along with what rollups are available.

The "Replication Latency" tab will show you if any of the cluster machines are out of date and how far behind they are. If you recently had to replace a machine or a disk drive, this would cause the host to need to be rebuilt or caught up.

The "Topology" tab relays what the layout of the ring looks like and how distributed the data should be.

The "Internals" tab shows internal application information, jobs, open sockets, and timers. This data is used by Support when troubleshooting issues.

### Performing Cluster Restarts

Any planned maintenance that requires restarting of a snowthd node (such as a host reboot, but also including a Hooper run that updates the platform/snowth package) should be performed with care to let the cluster "settle" after each node restart. Restarting too many nodes too quickly can cause a cascade of additional work that will dramatically lengthen the time required to return to a normal operating state and may adversely impact the availability of the entire cluster.

**Note:**
> The general rule of thumb is to allow about two minutes between Hooper runs or any other disruptive maintenance on {{{data_storage machines}}}, subject to observation of the replication latency described below.

Watch the "Replication Latency" tab of the Operations Dashboard during the restart process, noting the restarted node's lag relative to the others. It normally takes 30-60 seconds for the cluster to settle after a single node restart, but this may vary depending on the ingestion rate (how busy your cluster is). Do not restart the next node until the replication latency of the restarted node returns to green relative to all the other nodes.

## Snowth ZFS Condensing

Circonus Engineering has identified an issue that stems from the specific write workload placed on the system by the Data Storage application (Snowth). Over time, this may lead to severely reduced write performance and cluster instability. This issue primarily affects Data Storage clusters operating on OmniOS r151006, but may also arise on r151014 depending on the cluster node size and volume of incoming metrics.

To prevent this issue from occurring, Circonus recommends that cluster operators periodically rewrite the numeric storage portion of the Snowth data on each cluster node. This condenses the space allocations and prevents the performance problem.

**Note:**
> This maintenance should be performed every 6 months, or until Circonus Engineering devises a permanent fix in Snowth to prevent this issue. It should be performed on one node at a time, allowing the node to completely recover and catch up with its peers before continuing to the next node. The procedure is described below.

While this maintenance is being performed on a node, it will be unavailable to receive new incoming metric data. Metric data destined for this node will be journaled on other nodes in the cluster and replayed to it when it is returned to service.

If you have any questions concerning this issue, please contact Circonus Support (support@circonus.com).

### Reallocation Procedure

These steps are to be performed by a privileged user, and due to the length of time some commands take, it is recommended to use tmux or screen.

**NOTE:**
> If Snowth is running in a non-global zone, then this procedure should be performed within that zone.

 1. Stop the snowth service.
```
svcadm disable snowth
```

 1. Identify the numeric metric data dataset. Hereafter, this will be referred to as `<snowth-data>`. Use the actual dataset in the commands below.
```
zfs list -H -o name /snowth/data
```

 1. Create a source snapshot.
```
zfs snapshot <snowth-data>@condense
```

 1. Estimate the stream send size. Note both the size and the unit of measure (typically, 'G' or 'T'). This is only an estimate; the actual send size may be slightly larger.
```
zfs send -nv <snowth-data>@condense
```

 1. Perform the send/recv operation, using `/usr/bin/pv` (pipe-viewer) to measure progress. Here, `<size>` is the number from the stream estimate plus the unit of measure, such as "900G" (the version of pv on OmniOS r151006 does not support 'T' as a unit, so convert the value to gigabytes and use 'G'). pv will display, from left to right, a count of bytes transferred, an elapsed timer, the current transfer rate, the average transfer rate since start, and an ETA for completion.
```
zfs send -p <snowth-data>@condense | \

 pv -r -a -b -t -e -s <size> -B 512m | \

 zfs recv <snowth-data>-new
```

 1. Destroy the source snapshot.
```
zfs destroy <snowth-data>@condense
```

 1. Destroy the old dataset.
```
zfs destroy <snowth-data>
```

 1. Note that even though the previous command returns quickly, the actual freeing of data from the pool takes much longer and happens in the background. Do not proceed to the next step until the old data is finished being freed. You can monitor the freeing process with this command:
```
zpool get freeing
```

 1. When the above command reports 0 data freeing for the pool containing the Snowth datasets, rename the new dataset to the old name.
```
zfs rename <snowth-data>-new <snowth-data>
```

 1. Destroy the destination snapshot.
```
zfs destroy <snowth-data>@condense
```

 1. Run Hooper in its maintenance mode to make sure all settings are correct and then enable snowth.
```
/opt/circonus/bin/run-hooper -m
```

 1. Monitor the replay process using the Snowth Operations Dashboard. The node that has just been condensed will be receiving journal batches from all the other nodes. When its replication latency relative to all other nodes has returned to green, it is safe to proceed to the next node in the cluster.

## Delete Sweep Snowth API

Delete Sweep is a procedure that allows users to quickly remove large amounts of data from storage using the API. This procedure utilizes the following endpoint:

```
/sweep_delete
```

This endpoint can be invoked with the DELETE or the GET method. When invoked with the DELETE method, a maintenance delete will be started in a background thread on the node on which it is run. If invoked with the GET method, it will report the current maintenance delete status of the node. Each method is described below.

#### DELETE Method

This method kicks off a maintenance deletion. A timestamp is provided via header, along with a (comma separated) list of rollups for NNT and histogram data and a list of data types to clean up (NNT, text, or histogram). This list of data types defaults to nnt if none is specified. Headers include:

 * X-Snowth-Delete-Time: <timestamp> - (Required) All data up to this timestamp will be deleted across the system. For example, X-Snowth-Delete-Time: 1451606400 would delete all data before Jan. 1, 2016 GMT. The specified time must be before the current time. The call will fail if this is not provided.

 * X-Snowth-Delete-Data-Types: <data types> - (Optional) This header takes a comma-separated list of data types to delete. Valid values are "nnt", "text", and "histogram". For example, X-Snowth-Delete-Data-Types: nnt,text would delete the NNT and Text data up to the timestamp specified by the X-Snowth-Delete-Time header. Histogram data would not be touched. If this header is omitted, deleted values will default to NNT only.

 * X-Snowth-NNT-Delete-Rollups: <rollups> - (Optional) This header takes a comma-separated list of rollups to delete for NNT data. For example, X-Snowth-NNT-Delete-Rollups: 60,300 would delete data from the 60 and 300 rollups. Any other rollups on the system would not be touched. If this header is omitted, all rollups will be deleted.

 * X-Snowth-Histogram-Delete-Rollups - (Optional) This header takes a comma-separated list of rollups to delete for Histogram data. For example, X-Snowth-Histogram-Delete-Rollups: 60,300 would delete data from the 60 and 300 rollups. Any other rollups on the system would not be touched. If this header is omitted, all rollups will be deleted.

#### GET Method

This method takes no headers. It will return a JSON object that says if a maintenance run is running and what data it is currently working on.

Example output:

```
{

 "nnt": {

 "running":"true",

 "current_uuid":"1c2a1980-d3f2-4fcb-b24f-85a4a3b8cdd4",

 "timestamp":"1443110236",

 "rollups":"all"

 },

 "text": {

 "running":"true",

 "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-testmetric",

 "timestamp":"1443110236",

 },

 "histogram": {

 "running":"true",

 "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-BP",

 "timestamp":"1443110236",

 "rollups":"all"

 }

}
```

## Troubleshooting

### Repairing Corrupt LevelDB Data Stores

On occasion, a Snowth LevelDB database may become corrupted. Snowth has the capability to correct itself when this happens.

You should be able to determine which log is corrupted by looking at the errorlog for Snowth (usually in /snowth/logs/errorlog). It will tell you what has been corrupted. To fix it, follow the instructions below.

#### 1. Disable snowthd.

Before you start, you will need to disable snowthd with the following command:
```
sudo svcadm disable snowth
```

#### 2a. Correct corrupted text data.

There are two DBs that can become corrupted in the text db - the metrics store (a list of metrics) and the changelog (all of the different text values for a metric).

To correct the metrics store, run the following:

```
sudo /opt/circonus/sbin/snowthd -u nobody -g nobody \

 -r text/metrics \

 -i <id of snowth node in topology> \

 -c /opt/circonus/etc/snowth.conf \

 -D
```

To correct the changelog, run the the following:

```
sudo /opt/circonus/sbin/snowthd -u nobody -g nobody \

 -r text/changelog \

 -i <id of snowth node in topology> \

 -c /opt/circonus/etc/snowth.conf \

 -D
```

#### 2b. Correct corrupted histogram data.

For histogram data, the metrics db (a list of all available histogram metrics) or the actual data (which is stored based on the period) can become corrupted.

To fix the metrics database, run the following:

```
sudo /opt/circonus/sbin/snowthd -u nobody -g nobody \

 -r hist/metrics \

 -i <id of snowth node in topology> \

 -c /opt/circonus/etc/snowth.conf \

 -D
```

To fix the actual data, run the following:

```
sudo /opt/circonus/sbin/snowthd -u nobody -g nobody \

 -r hist/<period> \

 -i <id of snowth node in topology> \

 -c /opt/circonus/etc/snowth.conf \

 -D
```

#### 3. Renable snowthd.

Once finished, you will need to renable and clear snowthd with the following commands:

```
sudo svcadm enable snowth

sudo svcadm clear snowth
```


### Reconstituting a Data Storage Node

For instructions, refer to the section "[wiki:OperationManual/ReconstitutingaSnowthnode Reconstituting a Data Storage node]". This procedure is only used in circumstances where the node's data is completely unrecoverable. Always contact Circonus Support (support@circonus.com) before attempting these procedures.
