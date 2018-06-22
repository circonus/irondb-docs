# Deleting Text Data for a Check

This API call is for deleting text data from the IRONdb cluster for an entire check. It will remove data from the beginning of time up until the time provided by the user for every text metric that is part of the given check UUID.

This call will always return an empty array.

## Description

### URI

`/text/check/<uuid>`

### Method

DELETE

### Inputs

 * `uuid` : The UUID of the check.

### Headers

 * `X-Snowth-Delete-Time: <end>` (required)
   * `end` The end timestamp for the delete operation. All data from before this specified time is deleted. Time is represented in seconds since the epoch.
 * `X-Snowth-Full-Delete: <value>` (optional)
   * `value` Determines whether the delete operation is local to the receiving node (0) or journaled to all other nodes as well (1). The default, if not specified, is 0 (local-only delete).

## Examples

```
curl -X DELETE \
     -H 'X-Snowth-Delete-Time: 1527811200' \
     http://127.0.0.1:8112/text/check/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12
```

In this example:

 * `text` : This tells the system that text data will be removed.
 * `check` : This tells the system that text data will be removed for an entire check.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : Check UUID
 * `1527811200` : Delete all data for this check before this time

### Output

```
[]
```
