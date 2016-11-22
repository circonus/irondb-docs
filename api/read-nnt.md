Retrieving Numeric Data
=======================

This API call is for retrieving numeric data from the snowth cluster. It
will return an array with all the timestamps from the time given, along
with the attendant data.

Data will be returned in an array of tuples. Each tuple will contain a
timestamp and the value that was requested. If "all" data is requested,
the value returned is a hash with the name of each value and the value
itself.

Description of tuples
---------------------

`URI:`

:   /read/&lt;start&gt;/&lt;end&gt;/&lt;period&gt;/&lt;uuid&gt;/&lt;type&gt;/&lt;metric&gt;

`Method:`

:   GET

`Inputs:`

:   

    *start*

    :   The start time from which to pull data, represented in seconds
        since the epoch. This value is inclusive (data for the given
        start time will be pulled).

    *end*

    :   The end time up to which data is pulled, represented in seconds
        since the epoch. This value is exclusive (data up to, but not
        including, the given end time will be pulled).

    *period*

    :   The period, in seconds, for which to get data rollups.

    *uuid*

    :   The UUID of the check to which the metric belongs.

    *type*

    :   The type of data for which to pull results. Possible values for
        this input are as follows:

    :   

        *count*

        :   The number of data points received for the metric over the
            specified time period.

        *average*

        :   The average value for the metric over the specified
            time period.

        *derive*

        :   The derivative value for the metric over the specified
            time period.

        *counter*

        :   The counter value for the metric over the specified
            time period.

        *average\_stddev*

        :   The standard deviation of the average value for the metric
            over the specified time period.

        *derive\_stddev*

        :   The standard deviation of the derivative value for the
            metric over the specified time period.

        *counter\_stddev*

        :   The standard deviation of the counter value for the metric
            over the specified time period.

        *derive2*

        :   The second-order derivative value for the metric over the
            specified time period.

        *counter2*

        :   The second-order counter value for the metric over the
            specified time period.

        *derive2\_stddev*

        :   The standard deviation of the second-order derivative value
            for the metric over the specified time period.

        *counter2\_stddev*

        :   The standard deviation of the second-order counter value for
            the metric over the specified time period.

        *all*

        :   All of the above data.

    *metric*

    :   The name of the metric for which to pull data.

Examples
--------

This example uses

    /read/1380000000/1380000600/300/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/average/example

In this example:

*read*

:   This is the command to read data from the server.

*1380000000*

:   This is the Start Time (September 24, 2013, 05:20:00 GMT).

*1380000600*

:   This is the End Time (September 24, 2013, 05:30:00 GMT).

*300*

:   This is the Period (300 Seconds or 5 Minutes).

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*average*

:   This is the Metric Type.

*example*

:   This is the Metric Name.

`Output:`

    [[1380000000,50],[1380000300,60]]

This example uses

    /read/1379998800/1380009600/3600/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/all/example

In this example:

*read*

:   This is the command to read data from the server.

*1379998800*

:   This is the Start Time (September 24, 2013, 05:00:00 GMT).

*1380009600*

:   This is the End Time (September 24, 2013, 08:00:00 GMT).

*3600*

:   This is the Period (3600 Seconds or 1 Hour).

*6f6bdc73-2352-4bdc-ab0e-72f66d0dee12*

:   This is the UUID.

*all*

:   This is the Metric Type. "All" includes count, value, stddev,
    derivative, derivative\_stddev, counter, counter\_stddev,
    derivative2, derivate2\_stddev, counter2, and counter2\_stddev.

*example*

:   This is the Metric Name.

`Output:`

    [[1379998800,{"count":60,"value":10,"stddev":0,"derivative":0,"derivative_stddev":0,
    "counter":0,
    "counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,
    "counter2_stddev":0}],
    [1380002400,{"count":60,"value":10,"stddev":0,"derivative":0,"derivative_stddev":0,
    "counter":0,
    "counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,
    "counter2_stddev":0}],
    [1380006000,{"count":60,"value":10,"stddev":0,"derivative":0,"derivative_stddev":0,
    "counter":0,
    "counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,
    "counter2_stddev":0}]]
          
