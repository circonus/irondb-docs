# Writing Histogram Data

*This is legacy endpoint.  It is recommended to use the [Raw Data](./write-raw.html) submission endpoint to submit histogram data.*

This API call is for writing histogram data into the IRONdb cluster. The data will be sent as a JSON object containing the data to be added to the cluster.  

## Description

### URI

`/histogram/write`

### Method

PUT | POST

### JSON Format

 * `metric` : The name of the metric for which data is added.
 * `id` : The UUID of the check for the metric for which data is added.
 * `offset` : The timestamp, represented in time since the epoch, for which data is added.
 * `period` : The period for which to add the histogram data. Typically, this will be the smallest histogram period configured on the IRONdb cluster.
 * `histogram` : A base64-encoded compressed representation of the histogram data for this time period.

## Examples

The following example uses a file, `data.json`, containing the JSON object
below and posts it to an IRONdb node.

```
curl -X POST \
     -d @data.json \
     http://127.0.0.1:8112/histogram/write
```

The example JSON object below will add data to the IRONdb cluster for two
histogram metrics, named "example1" and "example2". The data will be added at
offset 1408724400 (August 22, 2014, 12:20:00 GMT).

`data.json` contents:

```json
[
  {
    "offset": 1408724400,
    "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e",
    "metric": "example1",
    "period": 60,
    "histogram": "AAA="
  },
  {
    "offset": 1408724400,
    "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e",
    "metric": "example2",
    "period": 60,
    "histogram": "AAA="
  }
]
```
