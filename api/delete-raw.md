# Deleting Raw Data for a Metric

This API call is for deleting raw data from an IRONdb node for a specific metric or for a set of metrics (when wildcards are specified). It will remove raw data which has not been rolled up from the beginning of time up until the end time provided by the user for that metric(s).

When used for deletion of a single metric, this call will return an empty array on success. If there is an error, it will return a JSON object with the error.

When used with wildcards, this call always returns a JSON object which describes the matching metrics and the potential or actual results of deletion for each metric.  Explicit confirmation is required in order to actually trigger the deletion, allowing the user to first issue the API call without confirmation to review what would actually be deleted (and hopefully avoid accidentally deleting more than intended).

Deletion is currently only supported on a single node per API call.  To delete data from the entire cluster, issue the same API call to each node.

## Description

### URI

`/raw/<uuid>/<metric>`

-OR-

`/raw/<uuid>/<metric_pattern_including_wildcards>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric from which data is deleted.
 * `metric_pattern_including_wildcards` : A metric naming pattern string including wildcards.

### Headers

 * `x-snowth-delete-time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data from before this specified time is deleted (if wildcards are specified, data will only get located and deletion will not occur unless confirmation is given). Time is represented in seconds since the epoch.

Used with wildcards only:
 * `x-snowth-account-id: <account_id>` (required)
   * `account_id` The account to be searched using the wildcard pattern
 * `x-snowth-confirm-delete: <0 or 1>` (optional, must be present and set to 1 to actually confirm and process with deletion)

## Single Metric Example

```
curl -X DELETE \
     -H 'x-snowth-delete-time: 1527811200' \
     http://127.0.0.1:8112/raw/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `raw` : This tells the system that raw data will be removed.
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
     http://127.0.0.1:8112/raw/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/multiple_example*
```

In this example:

 * `raw` : This tells the system that raw data will be removed.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `multiple_example*` : Metric name pattern including wildcard
 * `1527811200` : Delete all data for matching metrics before this time
 * `1234` : Delete data only for the given account id
 * `1` : Confirm to actually commit to the deletion (recommend omitting this header at first, to examine what will be deleted first)

### Sample Output for Wildcard Metric Example

```
[ {"metric_name":"multiple_example_cpuutil_server1","delete_result":"not local","payload":""},
  {"metric_name":"multiple_example_cpuutil_server2","delete_result":"deleted","payload":""},
  ...
]
```
