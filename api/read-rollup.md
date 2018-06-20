Retrieving Rollup Data
======================

This API call is for retrieving numeric data from the IRONdb cluster with fine grained control over the rollup_span. It will return an array with all the timestamps from the time given, along with the attendant data.

Data will be returned in an array of tuples. Each tuple will contain a timestamp and the value that was requested. If "all" data is requested, the value returned is a hash with the name of each value and the value itself.

**URI:** `/rollup/<uuid>/<metric_name>?start_ts=<start>&end_ts=<end>&rollup_span=<milliseconds>&get_engine=(dispatch|nnt|recalc)&type=<see below>`

**Method:** GET

**Inputs:**

  *metric*      : The name of the metric for which to pull data (url encoded).
  
  *uuid*        : The UUID of the check to which the metric belongs.
  
  *start_ts*    : The start time from which to pull data, represented in seconds.milliseconds since the unix epoch. 
  
  *end_ts*      : The end time up to which data is pulled, represented in seconds.milliseconds since the unix epoch. 
  
  *rollup_span* : The granularity of the rollup with a units suffix (`s` for seconds, `ms` for milliseconds.  See example below)
  
  *get_engine*  : 
  
   * `dispatch` means - read first from NNT and then fill in with recalculated raw data.
   * `nnt` means - behave like the [`/read`](read-nnt.md) endpoint, i.e. only read already rolled up data.
   * `recalc` means - read raw data and generate rollups on the fly.
  
  
  *type*        :   The type of data for which to pull results. Possible values for this input are as follows:

  * *count*   :   The number of data points received for the metric over the specified time period.
  * *average* :   The average value for the metric over the specified time period.
  * *derive* :   The derivative value for the metric over the specified time period.
  * *counter* :   The counter value for the metric over the specified time period.
  * *average_stddev* :   The standard deviation of the average value for the metric over the specified time period.
  * *derive\_stddev* :   The standard deviation of the derivative value for the metric over the specified time period.
  * *counter\_stddev* :   The standard deviation of the counter value for the metric over the specified time period.
  * *derive2* :   The second-order derivative value for the metric over the specified time period.
  * *counter2* :   The second-order counter value for the metric over the specified time period.
  * *derive2_stddev* :   The standard deviation of the second-order derivative value for the metric over the specified time period.
  * *counter2\_stddev* :   The standard deviation of the second-order counter value for the metric over the specified time period.
  * *all* :   All of the above data.
  
> A note on the `start_ts` and `end_ts` parameters:

The format is `<seconds since epoch>.<milliseconds>`.  Although this appears to be a decimal number it is not.
12345.6 does *not* mean 12345 seconds and 600 milliseconds it is an illegal format.  The `<milliseconds>` portion of the
timestamp must always be 3 digits to represent values from 000 to 999.
  
If `type` is omitted, the *average* is returned for each period.

Examples
---------

```
/rollup/fc85e0ab-f568-45e6-86ee-d7443be8277d/online?start_ts=1529509020.000&end_ts=1529509200.000&rollup_span=60000ms&type=all&get_engine=recalc"
```

*rollup*     : The rollup command

*fc85e0ab-f568-45e6-86ee-d7443be8277d* : The UUID of the check

*online* : The name of the metric to retrieve data for

*start_ts=1529509020.000* : 2018-06-20T15:37:00+00:00 and zero milliseconds

*end_ts=1529509225.000* : 2018-06-20T15:40:00+00:00 and zero milliseconds

*rollup_span=60000ms* : Rollup at 60000 milliseconds

*type=all* :  Get all rollup fields

*get_engine=recalc* : Recalculate the rollup from the raw database

**Output:**

```
   [
     [1529509020,{"count":1,"value":0,"stddev":0,"derivative":0,"derivative_stddev":0,"counter":0,"counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,"counter2_stddev":0}],
     [1529509080,{"count":1,"value":0,"stddev":0,"derivative":0,"derivative_stddev":0,"counter":0,"counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,"counter2_stddev":0}],
     [1529509140,{"count":1,"value":0,"stddev":0,"derivative":0,"derivative_stddev":0,"counter":0,"counter_stddev":0,"derivative2":0,"derivative2_stddev":0,"counter2":0,"counter2_stddev":0}]
   ]
```

Timestamps in the returned data will take `<seconds>.<milliseconds>` format if the rollup span requires that resolution.

