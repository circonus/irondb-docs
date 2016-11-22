Merging Numeric Data
====================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for merging histogram data into a Snowth node.

Raw binary data must be attached. The node that receives this will merge
this data into the local NNT data store.

Description of API call
-----------------------

`URI:`

:   /merge/&lt;period&gt;/&lt;uuid&gt;/&lt;metric&gt;

`Method:`

:   MERGE

`Headers:`

:   One header is required for this call. It is described below.

:   

    *X-Target-Topology*

    :   The target topology into which data is merged.

`Inputs:`

:   

    *period*

    :   The period, in seconds, that the data represents.

    *uuid*

    :   The UUID of the check to which the metric belongs.

    *metric*

    :   The name of the metric being merged.

Examples
--------

This example uses

    /merge/300/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example

    Header: "X-Target-Topology: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"

In this example:

*merge*

:   This is the command to perform a merge.

*300*

:   This is the Period (300 seconds or 5 minutes).

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*example*

:   This is the Metric Name.


