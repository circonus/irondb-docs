# Deleting Text Data for a Metric or a Set of Metrics

This API call is for deleting text data from the IRONdb cluster for a specific metric or for a set of metrics (when wildcards or a tag query are specified). It will remove text data from the beginning of time up until the end time provided by the user for that metric(s).

When used for deletion of a single metric, this call will return an empty array on success. If there is an error, it will return a JSON object with the error.

When used with wildcards or a tag query, this call always returns a JSON object which describes the matching metrics and the actions taken or errors received on the deletion.  For safety, explicit confirmation is required in the headers to actually force the data deletion.
A list of the possible result statuses for each metric and what they mean can be found [here](/data-deletion-statuses.md).

**It is highly recommended to perform the deletion API call without confirmation as a first step, in order to review what would actually be deleted (and hopefully avoid accidentally deleting more data than intended).**

Deletion of a single metric can optionally be journaled and replicated to all nodes using the `X-Snowth-Full-Delete` setting given for the headers below.  However, wildcard or tag query deletion is currently only supported on a single node per API call.  To use wildcards or a tag query to remove data across the cluster, issue the same API call to each node.

## Description

### URI

`/text/<uuid>/<metric>`

-OR-

`/text/<uuid>/<metric_pattern_including_wildcards>`

-OR-

`/text/tags?query=<query>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric from which data is deleted.
 * `metric_pattern_including_wildcards` : A metric naming pattern string including wildcards.
 * `query` : See [Tag Queries](/tag-queries.md) for more info on tag queries.

### Headers

 * `x-snowth-delete-time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data before this specified time is deleted (if wildcards or a tag query are specified, data will only be located and deletion will not occur unless confirmation is given). Time is represented in seconds since the epoch.

Used only with wildcards or tag query:
 * `x-snowth-account-id: <account_id>` (required)
   * `account_id` The account to be searched using the wildcard pattern
 * `x-snowth-confirm-delete: <0 or 1>` (optional, must be present and set to 1 to actually confirm and process the deletion)

Used only without wildcards or tag query:
 * `x-snowth-full-delete: <value>` (optional)
   * `value` Determines whether the delete operation is local to the receiving node (0) or journaled to all other nodes as well (1). The default, if not specified, is 0 (local-only delete). This setting means perform the delete across all nodes, and is not to be confused with the [full delete API](/api/delete-full.md).

## Single Metric Example

```
curl -X DELETE \
     -H 'X-Snowth-Delete-Time: 1527811200' \
     http://127.0.0.1:8112/text/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `text` : This tells the system that text data will be removed.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `example` : Metric name
 * `1527811200` : Delete all data for this metric before this time

### Sample Output for Single Metric Example

```
[]
```

## Wildcard Metric Example

```
curl -X DELETE \
     -H 'x-snowth-delete-time: 1527811200' \
     -H 'x-snowth-account-id: 1234' \
     -H 'x-snowth-confirm-delete: 1' \
     http://127.0.0.1:8112/text/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/multiple_example*
```

In this example:

 * `text` : This tells the system that text data will be removed.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `multiple_example*` : Metric name pattern including wildcard
 * `1527811200` : Delete all data for matching metrics before this time
 * `1234` : Delete data only for the given account id
 * `1` : Confirm to actually commit to the deletion (we highly recommend omitting this header at first, to examine what will be deleted)

### Sample Output for Wildcard Metric Example

```
[ {"metric_name":"multiple_example_cpuutil_server1","delete_result":"not local","payload":""},
  {"metric_name":"multiple_example_cpuutil_server2","delete_result":"deleted","payload":""},
  ...
]
```
