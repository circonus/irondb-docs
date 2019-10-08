# Retrieving Histogram Data

*This is legacy endpoint.  It is recommended to use the [Fetch](./fetch.md) endpoint for all data reads.*

This API call is for retrieving histogram data from the IRONdb cluster. It will return an array with all the timestamps from the time given, along with the attendant data.

Data will be returned in an array of arrays. Each sub-array will contain three elements: a timestamp, the period requested, and a JSON object representing the number of times that different values appeared in that time period.

## Description of arrays

### URI

`/histogram/<start>/<end>/<period>/<uuid>/<metric>`

### Method

GET

### Inputs

 * `start` : The start time from which to pull data, represented in seconds since the epoch. This value is inclusive (data for the given start time will be pulled).
 * `end` : The end time up to which data is pulled, represented in seconds since the epoch. This value is inclusive (data up to the given end time will be pulled).
 * `period` : The period, in seconds, for which to get data rollups.
 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric for which to pull data.

## Examples

```
curl http://127.0.0.1:8112/histogram/1380000000/1380000600/300/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/hist_example
```

In this example:

 * `histogram` : This is the command to read histogram data from the server.
 * `1380000000` : This is the Start Time (September 24, 2013, 05:20:00 GMT).
 * `1380000600` : This is the End Time (September 24, 2013, 05:30:00 GMT).
 * `300` : This the Period (300 Seconds or 5 Minutes).
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : This is the UUID.
 * `hist_example` : This is the Metric Name.

For the output of this example, assume that for every five-minute window, a different value between 1 and 5 was received at every one-minute interval.

### Example 1 Output

```json
[
  [
    1380000000,
    300,
    {
      "1": 1,
      "2": 1,
      "3": 1,
      "4": 1,
      "5": 1
    }
  ],
  [
    1380000300,
    300,
    {
      "1": 1,
      "2": 1,
      "3": 1,
      "4": 1,
      "5": 1
    }
  ]
]
```
