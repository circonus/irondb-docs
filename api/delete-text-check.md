Deleting Text Data for a Check
==============================

This API call is for deleting text data from the Snowth cluster for an
entire check. It will remove data from the beginning of time up until
the time provided by the user for every text metric that is part of the
given check UUID.

This call will always return an empty array.

Description of API call
-----------------------

`URI:`

:   /text/check/&lt;end&gt;/&lt;uuid&gt;

`Method:`

:   DELETE

`Inputs:`

:   

    *end*

    :   The end time. All data from before this specified time
        is deleted. Time is represented in seconds since the epoch.

    *uuid*

    :   The UUID of the check to delete.

Examples
--------

This example uses

    /text/check/1380000000/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12

In this example:

*text*

:   This is the command that tells the system that text data will
    be removed.

*check*

:   This is the command that tells the system that text data will be
    removed for an entire check.

*1380000000*

:   This is the End Time (September 24, 2013, 05:20:00 GMT).

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

`Output:`

    []
