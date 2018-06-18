# Deleting Text Data for a Metric

This API call is for deleting text data from the IRONdb cluster for a specific metric. It will remove data from the beginning of time up until the time provided by the user for that metric.

This call will return an empty array on success. If there is an error, it will return a JSON object with the error.

## Description

### URI

`/text/<uuid>/<metric>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric from which data is deleted.

### Headers

 * `X-Snowth-Delete-Time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data from before this specified time is deleted. Time is represented in seconds since the epoch.
 * `X-Snowth-Full-Delete: <value>` (optional)
   * `value` Determines whether the delete operation is local to the receiving node (0) or journaled to all other nodes as well (1). The default, if not specified, is 0 (local-only delete).

## Examples

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

### Output

```
[]
```
