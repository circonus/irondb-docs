# Retrieving Text Data

This API call is for retrieving text data from the Snowth cluster.

## Description

### URI

`/read/<start>/<end>/<uuid>/<metric>`

### Method

GET

### Inputs

 * `start` : The start time from which to pull data, represented in seconds
   since the epoch. This value is inclusive (data for the start time given will
   be pulled).
 * `end` : The end time from which to pull data, represented in seconds since
   the epoch. This value is exclusive (data up to, but not including, the given
   end time will be pulled).
 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric for which to pull data.

### Output

The response payload will contain a JSON array of tuples consisting of the timestamps (in millisecions since epoch) from the time the value was received, and the text value itself as a string.

The first value of the response array is an exception to this rule.
It will contain the timestamp of the start parameter, along with the text value at the start time.
This is done, so that the user has information about the initial state of the text metric for the request interval.

## Examples

```
curl http://127.0.0.1:8112/read/1380000000/1380000600/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/text_example
```

In this example:

 * `read` : This is the command to read data from the server.
 * `1380000000` : This is the Start Time (September 24, 2013, 05:20:00 GMT).
 * `1380000600` : This is the End Time (September 24, 2013, 05:30:00 GMT).
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : This is the Check UUID.
 * `text_example` : This is the Metric Name.

### Example 1 Output

```json
[[1380000000000,"test_value_at_the_beginning_of_the_requested_range"],[1380000000123,"test_value_2"]]
```
