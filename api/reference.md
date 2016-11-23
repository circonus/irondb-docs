**Name**

snowthd — Snowthd time-series database daemon.

**Synopsis**

snowthd \[-c _file_\] \[-i _uuid_\] \[-e\] \[-u _user_\] \[-g _group_\] \[-t _path_\] \[-d\] \[-D\] \[-l _logstream_\] \[-L

_logstream_\] \[-m\] \[-H\] \[-r text\/metrics\] \[-r text\/changelog\] \[-r hist\/metrics\] \[-r hist\/_period_\] \[-b text\]

\[-b hist\] \[-h\]

**Description**

**Snowthd **is the daemon that runs a single node of a distributed cluster of Snowth time-series database

nodes.

-u

Optional. Specifies a username or UID under which to operate.

-g

Optional. Specifies a group name or GID under which to operate.

-t

Optional. Specifies a path to chroot\(2\) for operation. Measures must be taken to ensure that log files

can be accessed in the chrooted environment.

-c

Specifies a configuration file to read. The initial configuration is read from this file and **write config**

commands will write to this file. The default location of the configuration file is noitd.conf in

the sysconfdir specified at build time.

-l

Marks the enabled bit for the specified log stream. This enables the stream even if it is disabled in the

configuration file. It must exist in the configuration file.

-L

Disables the enabled bit for the specified log stream. This disables the stream even if it is enabled in

the configuration file. It must exist in the configuration file.

-D

Instructs **noitd **to run in the foreground. Normally, noitd consists of a management process and a child

process that performs all actions. This prevents that scenario and runs only the child process attached

to the terminal. This is primarily for debugging purposes.

If this flag is specified more than once, **noitd **stays in the foreground, but the foreground process will

actually be the monitor process and the child will be forked off and respawned on unexpected failures.

-d

Enables the debug output facility despite configuration to the contrary.

-h

Displays the usage information.

\/histogram\/raw\/53276c66-3bb0-47fc-8474-cb2045d3da72

In this example:

_histogram_

This is the command to read histogram data from the server.

_raw_

This is the command to read raw data from the server.

_53276c66-3bb0-47fc-8474-cb2045d3da72_

This is the Node ID.

**Output:**

The output will be raw, binary output that represents all the histogram data on the target node that should

also be stored on the node with ID 53276c66-3bb0-47fc-8474-cb2045d3da72.



 

