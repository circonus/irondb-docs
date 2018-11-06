# Data Deletion APIs

---

Deletion APIs can be broken down into three categories, depending on the deletion scenario:
## Datatype Delete
[Datatype delete](data-deletion-datatype.md) refers to deleting data for a metric or a group of metrics - of a particular data type - up to a specified end time.
This is intended for metrics that will be remaining in use, but the user just wants to remove older data that is no longer needed.
## Sweep Delete
[Sweep delete](data-deletion-sweep.md) is used for deleting all metric data of all types up to a specified end time.
This is intended for removing all data older than the specified end time for all metrics of any data type, as part of storage hygiene and/or data retention policies.
## Full Delete
[Full delete](api/delete-full.md) allows the complete removal of all metric data and metadata for a metric or a group of metrics.
This is intended for completely removing a metric or a group of metrics and all of their data, when these metrics are no longer needed at all, were misnamed, or were experimental.
