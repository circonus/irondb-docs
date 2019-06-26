# Prometheus Ingestion

IRONdb has native endpoints for accepting remote write data from a Prometheus
installation.  Once the Prometheus module is enabled, data can be sent to IRONdb 
by setting the Prometheus `remote_write` endpoint to:

`http://irondbnode:8112/module/prometheus/write/<accountid>/<uuid>`

## Enable the Prometheus Module

IRONdb must be [configured](configuration.html) such that the Prometheus module is
enabled for reading or writing Prometheus data natively. Prometheus support is
activated by adding the following line:
```
<generic image="prometheus" name="prometheus"/>
```
to `/opt/circonus/etc/irondb-modules-site.conf` on each IRONdb node. This file
preserves local modifications across package updates. A [service
restart](operations.md#service-management) is required after changing
configuration.

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
account ID). Metric names can only be associated with one "account ID". This
allows separate client instances that completely segregate data.

## Writing Prometheus Data to IRONdb

To configure a Prometheus instance to write to IRONdb the Prometheus YAML
configuration file will need to be updated.  The `remote_write` section's `url`
field should be set to
`http://irondbnode:8112/module/prometheus/write/<accountid>/<uuid>`.  

This should look something like:
```
remote_write:
  - url: "https://irondbnode:8112/module/prometheus/write/1/321b704b-a8ff-44b7-8171-777dc49bc788"
```
