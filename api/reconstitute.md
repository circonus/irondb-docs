# Getting List of Metrics to Reconstitute

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call will return a list of metrics on the target node that
should be stored on the node identified by the node ID.

This will return a JSON object with a list of metrics. This format of
this object is defined below.

## Description

### URI

`/reconstitute/<nodeid>`

### Method

GET

### Inputs

 * `nodeid` : The UUID of the node that is requesting data.

### Output

 * `checkpoint` : The current JLOG checkpoint.
 * `metrics` : An array of metrics that should be read from the node. There are
   three types of entries: "text", "hist", and "nnt". Each of these are
   strings, with variable fields separated by "\\/". The fields in each one are
   described below.
   * `text` : The description of a text metric.
     * `type` : The type of data this line represents (text).
     * `id` : The UUID of the check that contains the metric.
     * `metric` : The name of the text metric that should be pulled,
       base64-encoded.
   * `hist` : The description of a histogram metric.
     * `type` : The type of data this line represents (histogram).
     * `id` : The UUID of the check that contains the metric.
     * `metric` : The name of the text metric that should be pulled,
       base64-encoded.
   * `nnt` : The description of an NNT (numeric) metric.
     * `type` : The type of data this line represents (nnt).
     * `rollup` : The rollup from which to pull the data.
     * `id` : The UUID of the check that contains the metric.
     * `metric` : The name of the text metric that should be pulled,
       base64-encoded.

## Examples

This example will pull a list of metrics that belong on the node given
in the URI from the node from which the API is pulling. The metrics in
this example are called `nnt_example`, `text_example`, and
`hist_example`, and they are base64-encoded.

```
curl http://127.0.0.1:8112/reconstitute/53276c66-3bb0-47fc-8474-cb2045d3da72
```

In this example:

 * `reconstitute` : This is the command to read reconstitute data from the server.
 * `53276c66-3bb0-47fc-8474-cb2045d3da72` : This is the Node ID

### Example 1 Output

```json
{
  "checkpoint":"00001111:000050a7",
  "metrics": [
    "text\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/dGV4dF9leGFtcGxl",
    "hist\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/aGlzdF9leGFtcGxl",
    "nnt\/60\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/bm50X2V4YW1wbGU=",
    "nnt\/300\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/bm50X2V4YW1wbGU=",
    "nnt\/3600\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/bm50X2V4YW1wbGU=",
    "nnt\/10800\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/bm50X2V4YW1wbGU=",
    "nnt\/86400\/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12\/bm50X2V4YW1wbGU="
  ]
}
```
