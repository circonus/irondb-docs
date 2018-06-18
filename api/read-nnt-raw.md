# Retrieving Raw Numeric Data

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for retrieving raw binary data for a numeric metric. It
will return the contents of the NNT file on the node for the metric.

## Description

### URI

`/raw/<rollup>/<uuid>/<filename>`

### Method

GET

### Inputs

 * `rollup` : The rollup for which to pull the raw NNT data. This must be an
   exact rollup value that is stored on the Snowth node.
 * `uuid` : The UUID of the check to which the metric belongs.
 * `filename` : The filename from which to pull the raw NNT data. The filename
   is the name of the metric, base64-encoded.

## Examples

This example will pull data for the five-minute rollup for the metric
named "example".

```
curl http://127.0.0.1:8112/raw/300/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/ZXhhbXBsZQ==
```

In this example:

 * `raw` : This is the command to read raw NNT data from the server.
 * `300` : This is the rollup (300 Seconds or 5 Minutes)
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : This is the Check UUID.
 * `ZXhhbXBsZQ==` : This is the filename (the word "example", base64-encoded)

### Example 1 Output

The output will be raw, binary output that represents all stored data
for the metric.
