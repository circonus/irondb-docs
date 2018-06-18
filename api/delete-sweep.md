# Deleting All Data Before A Date

This API call is for deleting any type of data from the IRONdb cluster prior to
a given date, irrespective of check or metric. This is known as a "sweep
delete", and is typically used to implement cluster-wide retention policies.

**This is an asynchronous, out-of-band administrative operation that does not
follow the normal consistency model of IRONdb.** The deletion only applies to
the node that received the request (deletions are not journaled to the cluster,
as normal data ingestion is.) If cluster-wide deletion of data is desired, this
call must be made to each cluster node individually.

If there is historical data being actively imported into the requested deletion
range, it can leave the delete operation "incomplete". A sweep delete will
never remove data that it should not, but it may appear that data that _should_
have been deleted may not be. This can leave the cluster in an inconsistent,
but not dangerous, state where another sweep delete may be required to clean up
stragglers.

The DELETE method will return an empty array upon success. If there is an
error, this call will return a JSON object with the error. Note that success
does not indicate completion of the delete operation, only that it has been
started. The order in which metric data is deleted is not deterministic. If a
sweep delete is in progress and another DELETE call is received, the running
sweep delete is cancelled, and the new operation begins.

The GET method will return a JSON object showing the status of a delete
operation. Each data type is listed with an attribute `running` set to the
string "true", and additional type-specific attributes indicating the current
data being operated on. If no operation is in progress, the `running` attribute
for each data type will be set to the string "false".

## Description

### URI

`/sweep_delete`

### Method

DELETE or GET

### Inputs

None. All options are provided by Headers (see below).

### Headers

> Headers are necessary when using the DELETE method, but not when obtaining
> status via the GET method.

 * `X-Snowth-Delete-Time: <new_epoch>` (required)
   * `new_epoch` The end timestamp for the delete operation. All data from
     before this specified time is deleted. Time is represented in seconds
     since the epoch.
 * `X-Snowth-Delete-Data-Types: <nnt,text,histogram>` (optional)
   * `nnt,text,histogram` The type of data (numeric, text, histogram) to remove
     may be specified, with multiple values comma-separated. If this header is
     not specified, only numeric data (nnt) will be removed.
 * `X-Snowth-NNT-Delete-Rollups: <span>[,<span>]` (optional)
   * `span` Specific NNT rollup spans may be removed (all spans will be removed
     if not specified), with multiple values comma-separated. The rollup
     span(s) are in seconds and must match one or more configured period values
     from the `<rollups>` stanza of `irondb.conf`.
 * `X-Snowth-Histogram-Delete-Rollups: <span>[,<span>]` (optional)
   * `span` Specific histogram rollup spans may be removed (all spans will be
     removed if not specified), with multiple values comma-separated. The
     rollup span(s) are in seconds and must match one or more configured period
     values from the `<histogram>` stanza of `irondb.conf`.

> Text metrics do not have the concept of rollups, so specifying the delete time
> and data type are sufficient to remove all text data prior to the given time.

## Examples

### Example 1

Delete data of all types and all rollup spans older than 2017-06-01

```
curl -X DELETE \
     -H 'X-Snowth-Delete-Time: 1496275200' \
     -H 'X-Snowth-Delete-Data-Types: nnt,text,histogram' \
     http://127.0.0.1:8112/sweep_delete
```

### Example 2

Delete all 1-minute numeric rollups older than 2017-06-01

```
curl -X DELETE \
     -H 'X-Snowth-Delete-Time: 1496275200' \
     -H 'X-Snowth-Delete-Data-Types: nnt' \
     -H 'X-Snowth-NNT-Delete-Rollups: 60' \
     http://127.0.0.1:8112/sweep_delete
```

### Example 3

Show the status of an active delete

```
curl http://127.0.0.1:8112/sweep_delete
```

#### Example 3 Output

```json
{
  "nnt": {
    "running":"true",
    "current_uuid":"1c2a1980-d3f2-4fcb-b24f-85a4a3b8cdd4",
    "timestamp":"1443110236",
    "rollups":"all"
  },
  "text": {
    "running":"true",
    "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-testmetric",
    "timestamp":"1443110236",
  },
  "histogram": {
    "running":"true",
    "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-BP",
    "timestamp":"1443110236",
    "rollups":"all"
  }
}
```
