# Operations

## Data Storage

The data storage service runs the irondb:default process, listening externally on port 8112 and locally on port 32322. Like the broker, this service has two processes: a child and a parent, which serves as watchdog for the child.

IRONdb is sensitive to CPU and IO limits. If either resource is limited, you can see child processes being killed off by the parents when they do not heartbeat on time.

Log files are located under /irondb/logs and include the following files:

 * accesslog

 * errorlog

The access logs are useful to verify activity going to the server in question. Error logs may contain debugging information for support personnel.

If the child process becomes unstable, verify that the host is not starved for resources (CPU, IO, memory). Hardware disk errors can also impact IRONdb's performance. To check for errors run:

```
iostat -zxne
```

You will see an "Errors" section to the right. If you begin to see hardware errors there, this could indicate a disk failure. If in doubt, contact Circonus Support (support@circonus.com) for assistance.

If instability continues, you can run IRONdb in the foreground. First, determine the node's ID by doing an:

```
ls /irondb/data/
```

Then, run the following as root:

```
/opt/circonus/sbin/irondbd -D -d \

 -u nobody \

 -g nobody \

 -c /opt/circonus/etc/irondb.conf \

 -i <nodeid from previous command>
```

Like the broker, running IRONdb in the foreground should allow you to capture a core dump, which Circonus Support can use to diagnose your problem.

### Operations Dashboard

IRONdb comes with built-in operational dashboard accessible from any data storage host on port 8112 in your browser, e.g., http://irondbhost:8112. This interface provides real-time information about the data storage cluster.

The "Overview" tab tells you the throughputs of data gets and puts along with what rollups are available.

The "Replication Latency" tab will show you if any of the cluster machines are out of date and how far behind they are. If you recently had to replace a machine or a disk drive, this would cause the host to need to be rebuilt or caught up.

The "Topology" tab relays what the layout of the ring looks like and how distributed the data should be.

The "Internals" tab shows internal application information, jobs, open sockets, and timers. This data is used by Support when troubleshooting issues.