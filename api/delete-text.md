Deleting Text Data for a Metric
===============================

This API call is for deleting text data from the IRONdb cluster for a specific metric. It will remove data from the beginning of time up until the time provided by the user for that metric.

This call will return an empty array on success. If there is an error, it will return a JSON object with the error.

Description of API call
-----------------------

**URI:**   /text/&lt;uuid&gt;/&lt;metric&gt;

**Method:**   DELETE

**Inputs:**

*uuid* :   The UUID of the check to which the metric belongs.

*metric* :   The name of the metric from which data is deleted.

**Headers:**

The end timestamp must be provided via a header:

X-Snowth-Delete-Time: &lt;end&gt;

*end* :   The end timestamp for the delete operation. All data from before this specified time is deleted. Time is represented in seconds since the epoch.

An optional header affects whether the delete operation is replicated to all
other nodes or confined to the node receiving the DELETE. The default is to
delete only locally on the receiving node. Set to `1` to cause the deletion to
be replicated.

X-Snowth-Full-Delete: 0

Examples
--------

This example uses

```
/text/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

*text* :   This tells the system that text data will be removed.

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12* :   This is the UUID.

*example* :   This is the Metric Name.

**Output:**

```
    []
```
