# Graphite Rendering

IRONdb has a graphite-web Storage Backend which makes the following Graphite Rendering seamless with an existing graphite-web installation. The Storage Backend requires graphite 0.10 or newer and can be obtained here:

https://github.com/circonus-labs/graphite-irondb

Follow the instructions in the README in that repo to install and utilize the IRONdb graphite storage backend.

That Storage Backend plugin simply utilizes the endpoints described below.

## Query Result Limits

All query results are subject to limits to control the number of results
returned. If not otherwise specified, queries will be limited to the first
10,000 results returned.

This limit may be changed by setting a request header,
`x-snowth-advisory-limit`, with one of the following values:
 * A positive integer representing the desired limit
 * -1 or "none" to remove the limit

If the header contains any other value or is not present, the default of 10,000
will be used.

## Searching for Metric Names

Graphite metrics can be fetched (rendered) from IRONdb using the following endpoints. Glob style wildcards are supported.

`http://<host:port>/graphite/<account_id>/<optional_query_prefix>/metrics/find?query=graphite.*`

This will return a JSON document with metrics matching the prefix: `graphite.` which terminate at that level.  Continuing on the example in graphite-ingestion.md, the above example would return the following:

```
[
        {"leaf": false, "name":"graphite.dev"},
        {"leaf": false, "name":"graphite.prod"}
]
```   

When a metric is a leaf node, `leaf` will be true and that metric will be queryable for actual datapoints.

The `optional_query_prefix` can be used to simplify metric names.  You can place any non-glob part of the prefix of a query into the `optional_query_prefix` and that prefix will be auto-prefixed to any incoming query for metric names. For example:

`http://<host:port>/graphite/1/graphite./metrics/find?query=*`

Will return:

```
[
        {"leaf": false, "name":"dev"},
        {"leaf": false, "name":"prod"}
]
```   

Note that the `optional_query_prefix` is omitted from the response json. You would use this feature to simplify all metric names in graphite-web or grafana and also to make IRONdb graphite metrics match metric names from an older time series system.

If you do not want to utilize the `optional_query_prefix` you can leave it off the URL:

`http://<host:port>/graphite/1/metrics/find?query=graphite.*`

```
[
        {"leaf": false, "name":"graphite.dev"},
        {"leaf": false, "name":"graphite.prod"}
]
```

## Searching for Tags

Graphite metrics can be fetched (rendered) from IRONdb using multi-dimensional tag queries.

`http://<host:port>/graphite/<account_id>/<optional_query_prefix>/tags/find?query=<tag query>`

This will return a JSON document with metrics matching the `<tag query>`.  Tag query syntax
is the same as supported by Graphite version >= 1.1.  See [Graphite Tag Querying](http://graphite.readthedocs.io/en/latest/tags.html#querying)

The syntax is:

    tag=spec    tag value exactly matches spec
    tag!=spec   tag value does not exactly match spec
    tag=~value  tag value matches the regular expression spec
    tag!=~spec  tag value does not match the regular expression spec
    
`http://<host:port>/graphite/1/tags/find?query=category1=value1`


```
[
        {"leaf": false, "name":"graphite.dev;category1=value1", "leaf_data": {...}},
        {"leaf": false, "name":"graphite.prod;category1=value1", "leaf_data": {...}}
]
```   


## Retrieving Datapoints

There are 2 methods for retrieving datapoints from IRONdb. A GET and a POST.

### GET

For retrieving an individual metric name, use:

`http://<host:port>/graphite/<account_id>/<optional_query_prefix>/series?start=<start_timestamp&end=<end_timestamp>&name=<metric_name>`

where `<start_timestamp>` and `<end_timestamp>` are expressed in unix epoch seconds, and `<metric_name>` is the originally ingested leaf node returned from the `/metrics/find` query above. `optional_query_prefix` 
follows the same rules as described in the prior section.

### POST

For fetching batches of time series data all at once, IRONdb provide a POST interface to send multiple names at the same time. To use this, POST a json document of `Content-type: application/json` to the following url:

`http://<host:port>/graphite/<account_id>/<optional_query_prefix>/series_multi`

The document format:

```
{
        "start": <start_timestamp>,
        "end" : <end_timestamp>,
        "names" : [ "graphite.dev.metric.one", "graphite.prod.metric.two"]
}
```

`optional_query_prefix` follows the same rules as the prior sections. If you provide an `optional_query_prefix` you would omit that portion of the metric name from the names in the JSON document. For example:

`http://<host:port>/graphite/1/graphite./series_multi`

The document format:

```
{
        "start": 0,
        "end" : 12345,
        "names" : [ "dev.metric.one", "prod.metric.two"]
}
```

