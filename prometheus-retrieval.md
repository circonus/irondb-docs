# Reading Prometheus Data from IRONdb

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

## Configure Remote Read

To configure a Prometheus instance to use IRONdb as a remote datasource, the Prometheus
YAML configuration file will need to be updated.  The `remote_read` section's
`url` field should be set to
`http://irondbnode:8112/module/prometheus/read/<accountid>/<uuid>`.

This should look something like:
```
remote_read:
  - url: "https://irondbnode:8112/module/prometheus/read/1/321b704b-a8ff-44b7-8171-777dc49bc788"
```
