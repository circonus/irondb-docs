Writing Numeric Data
====================

This API call is for writing NNT (numeric) data into the IRONdb cluster. It will send a JSON object containing the data to be added to the cluster.

Data should be added for the smallest rollup that exists on the IRONdb node. For example, if the smallest rollup on the cluster is 300 seconds (five minutes), five minute data should be added.

Description of JSON object
--------------------------

**URI:**   /write/nnt

**Method:**   PUT | POST

**JSON Format:**

*metric* :   The name of the metric for which data is added.

*id* :   The UUID of the check for the metric for which data is added.

*offset* :   The timestamp, represented in time since the epoch, for which data is added.

*count* :   The number of data points received for the metric over the specified time period.

*value* :   The average value for the metric over the specified time period.

*derivative* :   The derivative value for the metric over the specified time period.

*counter* :   The counter value for the metric over the specified time period.

*stddev* :   The standard deviation of the average value for the metric over the specified time period.

*derivative_stddev* :   The standard deviation of the derivative value for the metric over the specified time period.

*counter_stddev* :   The standard deviation of the counter value for the metric over the specified time period.

*parts* :   An optional array that contains the raw values that were used to calculate the values used above. The data is in the form of a tuple: the period (in seconds) that makes up the partial data, and an array of JSON objects that contains all of the fields above, except for "offset", "metric", and "id". The period value should be the values that are used to make up the smallest rollup. For example, if the smallest rollup is 300 seconds (five minutes) and that data was formed using 60 second (one minute) data, the "parts" data should have a period of 60, followed by five JSON objects describing the data at each interval.

This example uses

```
/write/nnt
```
The JSON object below will add data to the IRONdb cluster for two metrics, named "example1" and "example2". It assumes a smallest rollup value of 300 seconds and includes part data on 60 second intervals. The data will be added at offset 1408724400 (August 22, 2014, 12:20:00 GMT).

**Attached Text:**

```
    [
       { "derivative": 0, "counter": 0, "value": 100, "count": 5, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0, "offset": 1408724400, "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e", "metric": "example1", "parts": [ 60, [ { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 } ] ] },

       { "derivative": 0, "counter": 0, "value": 100, "count": 5, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0, "offset": 1408724400, "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e", "metric": "example2", "parts": [ 60, [ { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 }, { "derivative": 0, "counter": 0, "value": 100, "count": 1, "stddev": 0, "derivative_stddev": 0, "counter_stddev": 0 } ] ] }
    ]
```