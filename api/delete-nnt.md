Deleting Numeric Data for a Metric
==================================

This API call is for deleting numeric data from the IRONdb cluster for a specific metric. It will remove data from the beginning of time up until the time provided by the user for that metric. If the time given is greater than the current most recent data point in the file, the file will be removed.

This call will return an empty array upon success. If there is an error, this call will return a JSON object with the error.

Description of API call
-----------------------

**URI:**   /nnt/&lt;uuid&gt;/&lt;metric&gt;

**Method:**   DELETE

**Inputs:**   

*end* :   The end time. All data from before this specified time is deleted. Time is represented in seconds since the epoch.

*uuid* :   The UUID of the check to which the metric belongs.

*metric* :   The name of the metric from which to delete data.

The timestamp is provided via a header:

X-Snowth-Delete-Time: &lt;end&gt;

Examples
--------

This example uses

```
/nnt/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

*nnt* :   This is the command that tells the system that NNT data will be removed.

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12* :   This is the UUID.

*example* :   This is the Metric Name.

**Output:**

```
    []
```