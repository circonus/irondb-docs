# Deleting All Data for a Metric or a Set of Metrics

This API call is for deleting all of the data from an IRONdb node for a specific metric or for a set of metrics (when wildcards or a tag query are specified). It will remove data for the matching metric(s) throughout all timestamps and all rollups that have been provided by the user, no matter what the data type.  In addition, it will remove all record of the metric name(s) with their tags and metadata.  This call is intended for removing misnamed/experimental metrics or old metrics which are obsolete and can be safely removed.

When used for deletion of a single metric, this call will return an empty array on success. If there is an error, it will return a JSON object with the error.

When used with wildcards or a tag query, this call always returns a JSON object which describes the matching metrics and the actions taken or errors received on the deletion.
A list of the possible result statuses for each metric and what they mean can be found [here](/data-deletion-statuses.md).
For safety, explicit confirmation is required in the headers to actually force the data deletion.

**It is highly recommended to perform the deletion API call without confirmation as a first step, in order to review what would actually be deleted (and hopefully avoid accidentally deleting more data than intended).**

Deletion is currently only supported on a single node per API call.  To delete data from the entire cluster, issue the same API call to each node.

## Description

### URI

`/full/<uuid>/<metric>`

-OR-

`/full/<uuid>/<metric_pattern_including_wildcards>`

-OR-

`/full/tags?query=<query>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric from which data is deleted.
 * `metric_pattern_including_wildcards` : A metric naming pattern string including wildcards.
 * `query` : See [Tag Queries](/tag-queries.md) for more info on tag queries.

### Headers

Used only with wildcards or tag query:
 * `x-snowth-account-id: <account_id>` (required)
   * `account_id` The account to be searched using the wildcard pattern
 * `x-snowth-advisory-limit: <integer>|none` (optional, defaults to no limit if not present)
   * `integer` A positive integer specifying the number of matching results to
     delete. If the header is unset, or set to -1 or "none", the service will
     not limit the result set.
 * `x-snowth-confirm-delete: <0 or 1>` (optional, must be present and set to 1 to actually confirm and process the deletion)

## Single Metric Example

```
curl -X DELETE \
     http://127.0.0.1:8112/full/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `full` : This tells the system that full data and metadata will be removed for the specified metric.
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
     -H 'x-snowth-account-id: 1234' \
     -H 'x-snowth-confirm-delete: 1' \
     http://127.0.0.1:8112/full/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/multiple_example*
```

In this example:

 * `full` : This tells the system that all data and metadata for the matching metrics will be removed.
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
