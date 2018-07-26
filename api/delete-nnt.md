# Deleting Numeric Rollup Data for a Metric

This API call is for deleting numeric rollup data from the IRONdb cluster for a
specific metric. It will remove data from the beginning of time up until the
time provided by the user for that metric.

If using the deprecated NNT rollup storage format (as opposed to
[NNTBS](/configuration.md#nntbs)) and the time given is greater than the most
recent data point in the NNT file, the NNT file will be removed.

This call will return an empty array upon success. If there is an error, this
call will return a JSON object with the error.

## Description

### URI

`/nnt/<uuid>/<metric>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric from which to delete data.

### Headers

 * `X-Snowth-Delete-Time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data from before this specified time is deleted. Time is represented in seconds since the epoch.
 * `X-Snowth-Full-Delete: <value>` (optional)
   * `value` Determines whether the delete operation is local to the receiving node (0) or journaled to all other nodes as well (1). The default, if not specified, is 0 (local-only delete).

## Examples

```
curl -X DELETE \
     -H 'X-Snowth-Delete-Time: 1527811200' \
     http://127.0.0.1:8112/nnt/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `nnt` : This tells the system that numeric (NNT) data will be removed.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `example` : Metric name
 * `1527811200` : Delete all data for this metric before this time

## Output

```
[]
```
