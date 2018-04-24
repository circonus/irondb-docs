Cancelling A Sweep Delete
=========================

Cancels a running [sweep delete](/api/delete-sweep.md) operation.

The GET method will return a JSON object showing the status of a delete
operation. Each data type is listed with an attribute `cancelling` set to the
string "true", and additional type-specific attributes indicating the current
data being operated on. If no operation is in progress, the `running` attribute
for each data type will be set to the string "false".

Cancelling a sweep delete operation does not restore any data that was already
deleted. It merely stops wherever it was at the time the cancel request was
received. If the sweep delete was performed by mistake and you wish to recover
the data, you will need to [reconstitute the node](/rebuilding-nodes.md).

Description of API call
-----------------------

**URI:**   /sweep\_delete/cancel

**Method:**   GET

**Inputs:**   

None.

**Headers:**

None.

Examples
--------

```
curl http://127.0.0.1:8112/sweep_delete/cancel

{
  "nnt": {
    "running":"true",
    "cancelling":"true",
    "current_uuid":"1c2a1980-d3f2-4fcb-b24f-85a4a3b8cdd4",
    "timestamp":"1443110236",
    "rollups":"all"
  },
  "text": {
    "running":"true",
    "cancelling":"true",
    "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-testmetric",
    "timestamp":"1443110236",
  },
  "histogram": {
    "running":"true",
    "cancelling":"true",
    "current_metric":"1c27d35f-9dc6-447a-aa3f-2583bb88b591-BP",
    "timestamp":"1443110236",
    "rollups":"all"
  }
}
```
