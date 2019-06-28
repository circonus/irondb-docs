# Prometheus Integration

IRONdb supports remote write and read capabilities to provide long-term metric
storage for Prometheus deployments. One IRONdb cluster can support many
individual Prometheus instances.

The following pages detail how to write and read metric data:
* [Prometheus Remote Write](prometheus-ingestion.md)
* [Prometheus Remote Read](prometheus-retrieval.md)

## Load balancing requests

Both read and write requests to IRONdb can safely go to any node in an IRONdb
cluster.  To ensure high availability and distribute load, users are encouraged
to put a load balancer between the Prometheus nodes and the cluster.
