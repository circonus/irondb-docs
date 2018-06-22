# Writing Text Data

This API call is for writing text data into the IRONdb cluster. It sends a JSON object containing the data to be added to the cluster.

## Description

### URI

`/write/text`

### Method

PUT | POST

### JSON Format

 * `metric` : The name of the metric for which data is added.
 * `id` : The UUID of the check for the metric for which data is added.
 * `offset` : The timestamp, represented in time since the epoch, for which data added.
 * `value` : The text string to add to the IRONdb cluster.

## Examples

The following example uses a file, data.json, containing the JSON object below
and posts it to an IRONdb node.

```
curl -X POST \
     -d @data.json \
     http://127.0.0.1:8112/write/text
```

The JSON object below will add data to the IRONdb cluster for two text metrics,
named "textexample1" and "textexample2". The data will be added at offset
1408724400 (August 22, 2014, 12:20:00 GMT).

`data.json` contents:

```json
[
  {
    "offset": "1408724400",
    "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e",
    "metric": "textexample1",
    "value": "this_is_a_test"
  },
  {
    "offset": "1408724400",
    "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e",
    "metric": "textexample2",
    "value": "this_is_also_a_test"
  }
]
```
