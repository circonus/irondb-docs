Deleting Text Data for a Metric
===============================

This API call is for deleting text data from the Snowth cluster for a
specific metric. It will remove data from the beginning of time up until
the time provided by the user for that metric.

This call will return an empty array on success. If there is an error,
it will return a JSON object with the error.

Description of API call
-----------------------

`URI:`

:   /text/&lt;end&gt;/&lt;uuid&gt;/&lt;metric&gt;

`Method`

:   DELETE

`Inputs:`

:   

    *end*

    :   The end time. All data from before this specified time
        is deleted. Time is represented in seconds since the epoch.

    *uuid*

    :   The UUID of the check to which the metric belongs.

    *metric*

    :   The name of the metric from which data is deleted.

Examples
--------

This example uses

    /text/1380000000/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/text_example

In this exampl:

*text*

:   This is the command that tells the system that text data will
    be removed.

*1380000000*

:   This the End Time (September 24, 2013, 05:20:00 GMT).

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*example*

:   This is the Metric Name.

`Output:`

    []
