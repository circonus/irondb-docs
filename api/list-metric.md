# List All Metrics Stored on a Node

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call will retrieve information for every metric stored on a
particular node.

Data will be returned as an array of JSON objects. The fields in each
object are described below.

## Description

### URI

`/list/metric`

### Method

GET

### Output

 * `id` : The UUID of the check that contains the metric.
 * `metric` : The name of the metric being stored.
 * `type` : The type of the metric (numeric, text, or histogram).

## Examples

```
curl http://127.0.0.1:8112/list/metric
```

### Example 1 Output

```json
[
   {"id":"07658cc8-2b36-4819-ab76-50913ae02384","metric":"numeric_example","type":"numeric"},
   {"id":"07658cc8-2b36-4819-ab76-50913ae02384","metric":"text_example","type":"text"},
   {"id":"07658cc8-2b36-4819-ab76-50913ae02384","metric":"histogram_example","type":"histogram"},
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"numeric_example","type":"numeric"},
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"text_example","type":"text"},
   {"id":"821b187a-07fa-45e1-b727-d5b2b0e85993","metric":"histogram_example","type":"histogram"},
   {"id":"8a12a395-ad4c-4cc4-ae27-525f63e1fab5","metric":"numeric_example","type":"numeric"},
   {"id":"8a12a395-ad4c-4cc4-ae27-525f63e1fab5","metric":"text_example","type":"text"},
   {"id":"8a12a395-ad4c-4cc4-ae27-525f63e1fab5","metric":"histogram_example","type":"histogram"}
]
```
