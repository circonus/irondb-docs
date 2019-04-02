# Prometheus Ingestion

IRONdb has native endpoints for accepting remote write data from a Prometheus
installation.  Once the Prometheus module is enabled, data can be sent to IRONdb 
by setting the Prometheus remote_write endpoint to:

`http://irondbnode:8112/module/prometheus/write/<accountid>/<uuid>`

## Enabling Prometheus Module

IRONdb must be [configured](configuration.html) such that the Prometheus module is
enabled for reading or writing Prometheus data natively.  In the <modules> section of 
the IRONdb config, Prometheus support is activated by including the following stanza:
```
<generic image="prometheus" name="prometheus"/>
```
It is suggested this be placed inside of `/opt/circonus/etc/irondb-modules-site.conf`

## Namespacing

Prometheus data is not namespaced by nature.  This can create confusion if different
copies of Prometheus have identically named metrics.  Inside of IRONdb, we require 
that all data be namespaced under a UUID.  This UUID can be arbitrarily chosen or 
created using `uuidgen` on a typical unix system.  Each distinct set of 
Prometheus data should have its own UUID.   For high-availability in Prometheus it
is the recommended pratice to have two copies collecting the same data.  While these
two instances do not contain the same data, they do represent the same metrics, and so
should share a common UUID for their namespace.  One may wish to send both of these
instances into IRONdb where they simply become more samples in the given metric stream.

All metrics live under a numeric identifier (one can think of this like an
account_id). Metric names can only be associated with one "account_id". This
allows separate client instances that completely segregate data.

## Writing Prometheus Data to IRONdb

To configure a Prometheus instance to write to IRONdb the Prometheus YAML configuration 
file will need to be updated.  The remote_write section's `url` field should be set
to `http://irondbnode:8112/module/prometheus/write/<accountid>/<uuid>`.  

This should look something like:
```
remote_write:
  - url: "https://irondbnode:8112/module/prometheus/write/1/321b704b-a8ff-44b7-8171-777dc49bc788"
```

## Reading Prometheus Data from IRONdb

To configure a Prometheus instance to use IRONdb as a remote datasource, the Prometheus 
YAML configuration file will need to be updated.  The remote_read section's `url` field
should be set to `http://irondbnode:8112/module/prometheus/read/<accountid>/<uuid>`.

This should look something like:
```
remote_read:
  - url: "https://irondbnode:8112/module/prometheus/read/1/321b704b-a8ff-44b7-8171-777dc49bc788"
```

## Load balancing requests

Both read, and write requests to IRONdb can safely go to any node in an IRONdb cluster.  To ensure
high availability and distribute load, users are encouraged to put a load balancer between
the Prometheus nodes and the cluster.
