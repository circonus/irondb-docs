Retrieving Raw Numeric Data
===========================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for retrieving raw binary data for a numeric metric. It
will return the contents of the NNT file on the node for the metric.

Description of API call
-----------------------

`URI:`

:   /raw/&lt;rollup&gt;/&lt;uuid&gt;/&lt;filename&gt;

`Method:`

:   GET

`Inputs:`

:   

    *rollup*

    :   The rollup for which to pull the raw data. This must be an exact
        rollup value that is stored on the Snowth node.

    *uuid*

    :   The UUID of the check to which the metric belongs.

    *filename*

    :   The filename from which to pull the NNT data. The filename is
        the name of the metric, base 64 encoded.

This example will pull data for the five-minute rollup for the metric
named "example".

    /raw/300/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/ZXhhbXBsZQ==

In this example:

*raw*

:   This is the command to read raw data from the server.

*300*

:   This is the Rollup (300 Seconds or 5 Minutes)

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*ZXhhbXBsZQ==*

:   This is the Filename. (The word "example", base 64 encoded,
    becomes "ZXhhbXBsZQ==".)

`Output:`

The output will be raw, binary output that represents all stored data
for the metric.
