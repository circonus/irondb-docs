# Deleting Numeric Data for a Check

This API call is for deleting numeric data from the IRONdb cluster for an entire check. It will remove data from the beginning of time up until the time provided by the user for every numeric metric that is part of
the given check UUID.

If using the deprecated NNT rollup storage format (as opposed to
[NNTBS](/configuration.md#nntbs)) and the time given is greater than the most
recent data point in the NNT file, the NNT file will be removed.

The default behavior is deletion of data for all rollups, but it is possible also to specify particular rollups in the header given below, which may be used to remove data for specific rollup(s) which are not needed.

This call always returns a JSON object which describes the matching metrics and the actions taken or errors received on the deletion.  For safety, explicit confirmation is required in the headers to actually force the data deletion.  It is highly recommended to perform the deletion API call without confirmation as a first step, in order to review what would actually be deleted (and hopefully avoid accidentally deleting more data than intended).

Deletion is currently only supported on a single node per API call.  To delete data from the entire cluster, issue the same API call to each node.

## Description

### URI

`/nnt/<uuid>`

### Method

DELETE

### Inputs

 * `uuid` The UUID of the check.

### Headers

 * `x-snowth-delete-time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data before this specified time is deleted. Time is represented in seconds since the epoch.
 * `x-snowth-account-id: <account_id>` (required)
   * `account_id` The account to be searched using the wildcard pattern
 * `x-snowth-delete-rollups: <rollups>` (optional, if omitted the default is ALL rollups)
   * `rollups` The rollups that should be included in the deletion operation, separated by commas
 * `x-snowth-confirm-delete: <0 or 1>` (optional, must be present and set to 1 to actually confirm and process the deletion)

## Examples

```
curl -X DELETE \
     -H 'x-snowth-delete-time: 1527811200' \
     -H 'x-snowth-account-id: 1234' \
     -H 'x-snowth-confirm-delete: 1' \
     http://127.0.0.1:8112/nnt/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12
```

In this example:

 * `nnt` : This tells the system that nnt data will be removed.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `1527811200` : Delete all data for this check before this time

### Output

```
[ {"metric_name":"multiple_example_cpuutil_server1","delete_result":"not local","payload":""},
  {"metric_name":"multiple_example_cpuutil_server2","delete_result":"deleted","payload":""},
  ...
]
```
