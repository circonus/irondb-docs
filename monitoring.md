# Monitoring IRONdb

Each IRONdb node exposes a wealth of information about its internal operation.
There are two ways to obtain this data: pulling JSON from a REST stats
endpoint, or having IRONdb push its own stats into a particular account/check
using a loadable module. In both cases, the metrics exposed are the same.

The types of statistics available are described in the [Internal
Stats](/operations.md#stats) section of the Operations page.

The JSON endpoint is best for viewing live information, and
for long-term trending when IRONdb is part of a full Circonus Inside,
on-premises deployment. The internal monitor module is best suited to
long-term trending in standalone IRONdb deployments. Its metrics may be
retrieved using one of the type-specific [data retrieval
APIs](/data-retrieval-apis.md).

Both methods are described below.

## JSON

JSON-formatted metrics are available from two REST endpoints, each having two
format options:
* `http://<nodename or ip>:8112/stats.json` or `/stats.json?format=tagged`
* `http://<nodename or ip>:8112/mtev/stats.json` or `/mtev/stats.json?format=tagged`

The first endpoint provides application-level statistics, such as database
performance, replication latencies, topology info, etc. These are the same
metrics that are visible in the [UI Internals
tab](/operations.md#internals-tab) Stats pane under the `snowth.` namespace.

The second endpoint provides libmtev framework statistics, such as job queue
latencies, memory management, and REST API latencies. These are the same
metrics that are visible in the [UI Internals
tab](/operations.md#internals-tab) Stats pane under the `mtev.` namespace.

The format options are discussed below.

> Changing an existing check against the default format to tagged format, or
> vice versa, will result in different metric names, even though the data
> represented is the same.

### JSON Default Format

The default format for metric names is hierarchical. The broadest category of
statistics is the top level, descending to more specific sub-categories, and
finally listing individual metrics.

For example, the raw database PUT latency histogram metric
is represented in the default format as:

```json
{
  "db": {
    "raw": {
      "put`latency": {
        "_type": "h",
        "_value": [ (histogram values) ]
      }
    }
  }
}
```

which results in a metric named:

```
db`raw`put`latency
```

There are no tags in the default format.

### JSON Tagged Format

If provided the query string `format=tagged`, both endpoints will produce
metrics with [stream tags](/tags.md) instead of the hierarchy used in the
default format. The same metric from above is represented in tagged format as:

```json
{
  "latency|ST[app:snowth,db-impl:nom,db-type:raw,operation:put,snowth-node-id:(node-uuid),units:seconds]": {
    "_type": "h",
    "_value": [ (histogram values) ]
  }
}
```

which results in a metric named `latency` with tags indicating the database
type (raw) and the type of operation (put) that are encoded in the metric name
in the default format. There are additional tags for the node's UUID and a
"units" tag indicating what the metric's value represents. In this case it is
seconds.

## Monitor Module

The internal monitor module exports all of the same statistics (both
application and libmtev framework) as the JSON endpoints above. It records them
in the tagged format (described above) under a designated account ID and check
UUID. The module may be configured to store these metrics at intervals ranging
from 1 second to several minutes or more.

Metrics stored by the monitor module are replicated to additional nodes (if
any) in the same way as metrics ingested from outside.

The monitor module is not enabled by default. To enable it, add the following
configuration to `/opt/circonus/etc/irondb-modules-site.conf` then restart the
IRONdb service:

```xml
<generic image="monitor" name="monitor">
  <config>
    <uuid>00000000-0000-0000-0000-000000000000</uuid>
    <account_id>1</account_id>
    <period>60s</period>
  </config>
</generic>
```

> This file will preserve local edits across package updates.

Available configuration parameters:
 * `uuid` (required): The check UUID under which the module's metrics should be
   stored.
 * `account_id` (optional): The account ID with which to associate the module's
   metrics. Default: 1
 * `period` (optional): The collection period for metrics. Specified as an
   integer suffixed by one of `(ms|s|min|hr)`. Minimum value is 1 second.
   Default: 60s

The check UUID is an identifier for grouping the internal metrics together. It
is recommended that you choose a UUID that is different from any associated
with Graphite, Prometheus, or OpenTSDB listener configurations. This will
ensure that the internal metrics are not mixed in with your external time
series data. Likewise, `account_id` may be used as another level of
segregation, or you may choose to leave the metrics in the same account ID as
your other metrics.

### Viewing Monitor Metrics

To get a list of metrics recorded by the module, perform a
[tag query](/tag-queries.md) using the synthetic `__check_uuid` tag:

```
curl 'http://127.0.0.1:8112/find/<account_id>/tags?query=and(__check_uuid:<check_uuid>)'
```

The search results may be narrowed by including additional tags. In the
following example, we are looking for the latency of raw-database PUT
operations:

```
curl 'localhost:8112/find/1/tags?query=and(__check_uuid:d8c204ed-c2b6-4704-b6ec-f87787aad21f,db-type:raw,operation:put,__name:latency)'
```

which produces this result:

```json
[
  {
    "uuid": "d8c204ed-c2b6-4704-b6ec-f87787aad21f",
    "check_name": "irondb-monitor",
    "metric_name": "latency|ST[app:snowth,db-impl:nom,db-type:raw,operation:put,snowth-node-id:12c07a06-2662-4ceb-86a8-ccd05eef0f48,units:seconds]",
    "category": "reconnoiter",
    "type": "histogram",
    "account_id": 1
  }
]
```

The metric is reported to be a histogram, so using the [histogram read
API](/api/read-histogram.md) we can fetch some data for this metric. We need to
URL-encode the metric name since it contains some characters that are not
allowed in URLs.

```
curl 'localhost:8112/histogram/1557934740/1557934799/60/d8c204ed-c2b6-4704-b6ec-f87787aad21f/latency%7CST%5Bapp%3Asnowth%2Cdb-impl%3Anom%2Cdb-type%3Araw%2Coperation%3Aput%2Csnowth-node-id%3A12c07a06-2662-4ceb-86a8-ccd05eef0f48%2Cunits%3Aseconds%5D'
```

Result:

```json
[
  [
    1557934740,
    60,
    {
      "+75e-005": 1,
      "+79e-005": 2,
      "+82e-005": 2,
      "+83e-005": 1,
      "+84e-005": 1,
      "+86e-005": 2,
      "+88e-005": 1,
      "+89e-005": 3,
      "+90e-005": 1,
      "+92e-005": 2,
      "+93e-005": 2,
      "+95e-005": 1,
      "+10e-004": 11,
      "+11e-004": 7,
      "+12e-004": 7,
      "+13e-004": 10,
      "+14e-004": 8,
      "+15e-004": 5,
      "+16e-004": 12,
      "+17e-004": 15,
      "+18e-004": 5,
      "+19e-004": 5
    }
  ]
]
```

