# Retrieving Data Location In JSON Format

This API call retrieves a list of all of the nodes on which a metric resides.

Data will be returned as a JSON object. The format of this object is described below.

## Description

### URI

`/locate/json/<uuid>/<metric>`

### Method

GET

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric to locate.

### Output

 * `key` : The key used to locate the metric. It is in the form `<UUID>-<metric>`.
 * `location` : An array of JSON objects representing the nodes on which the
   data resides. The format of each object in the array is as follows:
   * `id` : The UUID of the node.
   * `address` : The IP Address of the node.
   * `port` : The port on which the node is listening.
   * `apiport` : The port on which the API is listening for the node.
   * `weight` : A value representing the relative preference weight of this
     node for metric ownership, compared to its peers.
   * `n` : The number of nodes in the topology on which data is stored.

## Examples

```
curl http://127.0.0.1:8112/locate/json/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `locate` : This is the command to locate a check/metric.
 * `json` : This is the command to read data in JSON format.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : This is the Check UUID.
 * `example` : This is the Metric.

### Example 1 Output

```json
{
  "key": "6f6bdc73-2352-4bdc-ab0e-72f66d0dee12-example",
  "location": [
    {
      "id": "1f846f26-0cfd-4df5-b4f1-e0930604e577",
      "address": "10.8.20.1",
      "port": 8112,
      "apiport": 8112,
      "weight": 32,
      "n": 2
    },
    {
      "id": "07fa2237-5744-4c28-a622-a99cfc1ac87e",
      "address": "10.8.20.4",
      "port": 8112,
      "apiport": 8112,
      "weight": 32,
      "n": 2
    }
  ]
}
```
