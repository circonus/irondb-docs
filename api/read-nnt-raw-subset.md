Retrieving Raw Numeric Subset Data
==================================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for retrieving raw binary data for a numeric metric. It
will return an NNT file for the specified subset of the numeric metric
specified.

Description of API call
-----------------------

`URI:`

:   /raw\_subset/&lt;uuid&gt;/&lt;filename&gt;/&lt;rollup&gt;/&lt;whence&gt;/&lt;num\_entries&gt;

`Method:`

:   GET

`Inputs:`

:   

    *uuid*

    :   The UUID of the check to which the metric belongs.

    *filename*

    :   The filename from which to pull the NNT data. The filename is
        the name of the metric, base 64 encoded.

    *rollup*

    :   The rollup for which to pull the raw data. This must be an exact
        rollup value that is stored on the Snowth node.

    *whence*

    :   The start time from which to pull data, represented in seconds
        since the epoch.

    *num\_datapoints*

    :   The number of datapoints after the "whence" time to pull.

This example will pull 10 datapoints for the five-minute rollup for the
metric named "example", starting at offset 1380000000.

    /raw_subset/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/ZXhhbXBsZQ==/300/1380000000/10

In this example:

*raw\_subset*

:   This is the command to read raw subset data from the server.

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*ZXhhbXBsZQ==*

:   This is the Filename. (The word "example", base 64 encoded,
    becomes "ZXhhbXBsZQ==".)

*300*

:   This is the Rollup (300 Seconds or 5 Minutes).

*1380000000*

:   This is the Whence Time (September 24, 2013, 05:20:00 GMT).

*10*

:   This is the Number of Datapoints to Pull.

`Output:`

The output will be raw, binary output that is a storage file for the
specified NNT data.
