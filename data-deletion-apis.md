# Data Deletion APIs

---

Deletion APIs can be broken down into three categories, depending on the deletion scenario:
## Deleting data for a metric or group of metrics of a particular data type up to a specified end time (datatype delete)
This is intended for metrics that will be remaining in use, but the user just wants to remove older data that is no longer needed. [Click here for details.](data-deletion-datatype.md)
## Deleting all metric data of all types up to a specified end time (sweep delete)
This is intended for removing all data older than the specified end time for all metrics of any data type, as part of storage hygeine and/or data retention policies. [Click here for details.](data-deletion-sweep.md)
## Deleting all metric data and metadata for a metric or group of metrics (full delete)
This is intended for completely removing a metric or a group of metrics and all of their data, when these metrics are no longer needed at all, were misnamed, or were experimental. [Click here for details.](api/delete-full.md)
