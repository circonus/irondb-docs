## Graphite Rendering

IRONdb has a graphite-web Storage Backend which makes the following seamless with an existing graphite-web installation.  The Storage Backend requires graphite 0.10 or newer and can be obtained here: https://github.com/circonus-labs/graphite-irondb

Follow the instructions in the README in that repo to install and utilize the IRONdb graphite storage backend.

That Storage Backend plugin simply utilizes the endpoints that follow.

Searching for metric names
==========================

Graphite metrics can be fetched (rendered) from IRONdb using the following endpoints.  Glob style
wildcards are supported.

`http://<host:port>/graphite/<account_id>/metrics/find?query=graphite.*`

This will return a JSON document with metrics matching the prefix: `graphite.` which terminate that that level.  
Continuing on the example in graphite-ingestion.md, the above would return:

```
[
        {"leaf": false, "name":"graphite.dev"},
        {"leaf": false, "name":"graphite.prod"}
]
```   

When a metric is a leaf node, `leaf` will be true and that metric will be queryable for actual datapoints.

Retrieving datapoints
=====================

There are 2 methods for retrieving datapoints from IRONdb.  A GET and a POST.

GET
---

`http://<host:port>/graphite/<account_id>/series?start=<start_timestamp&end=<end_timestamp>&name=<metric_name>`

Where `<start_timestamp>` and `<end_timestamp>` are expressed in unix epoch seconds and `<metric_name>` is
the originally ingested leaf node returned from the `/metrics/find` query above.

POST
----

For fetching batches of time series data all at once, IRONdb provide a POST interface to send multiple names 
at the same time.  To use this POST a json document of `Content-type: application/json` to the following
url:

`http://<host:port>/graphite/<account_id>/series_multi`

The document format:

```
{
        "start": <start_timestamp>,
        "end" : <end_timestamp>,
        "names" : [ "graphite.dev.metric.one", "graphite.prod.metric.two"]
}
```

