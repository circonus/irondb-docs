# List All Metrics for a Specific Check Stored on a Node

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call will retrieve information for every metric associated with
a specific check stored on a particular node.

Data will be returned as an array of JSON objects. The fields in each
object are described below.

## Description

### URI

`/list/metric/<uuid>`

### Method

GET

### Inputs

 * `uuid` : The UUID of the check from which to pull metrics.

### Output

 * `id` : The UUID of the check that contains the metric.
 * `metric` : The name of the metric being stored.
 * `type` : The type of the metric (numeric, text, or histogram).

## Examples

```
curl http://127.0.0.1:8112/list/metric/821b187a-07fa-45e1-b727-d5b2b0e85993
```

In this example:

 * `list` : This is the command to list data.
 * `metric` : This is the command to list metrics.
 * `821b187a-07fa-45e1-b727-d5b2b0e85993` : Check UUID

### Example 1 Output

```json
[
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"numeric_example","type":"numeric"},
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"text_example","type":"text"},
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"histogram_example","type":"histogram"}
]
```
