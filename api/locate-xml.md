# Retrieving Data Location In XML Format

This API call retrieves a list of all of the nodes on which a metric resides.

Data will be returned as an XML document. The format of this document is described below.

## Description

### URI

`/locate/xml/<uuid>/<metric>`

### Method

GET

### Inputs

 * `uuid` : The UUID of the check to which the metric belongs.
 * `metric` : The name of the metric to locate.

### Output

 * `<nodes>` : The Top-Level XML for the topology.
  * Attributes:
    * `n` : The number of nodes on which the data will be stored.
  * `<node>` : The container for all the information on a single node in the
    cluster. There will be up to x of these, where "x" is the number of nodes
    in the cluster.
    * Attributes:
      * `id` : The UUID of the node.
      * `address` : The IP Address of the node.
      * `port` : The port on which the node is listening.
      * `apiport` : The port on which the API is listening for the node.
      * `weight` : A value representing the relative preference weight of this
        node for metric ownership, compared to its peers.

## Examples

```
curl http://127.0.0.1:8112/locate/xml/6f6bdc73-2352-4bdc-ab0e-72f66d0dee12/example
```

In this example:

 * `locate` : This is the command to locate a check/metric.
 * `xml` : This is the command to read data in XML format.
 * `6f6bdc73-2352-4bdc-ab0e-72f66d0dee12` : This is the Check UUID.
 * `example` : This is the Metric.

### Example 1 Output

```xml
<nodes n="2">
  <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
           address="10.8.20.1"
           port="8112"
           apiport="8112"
           weight="32"/>
  <node id="07fa2237-5744-4c28-a622-a99cfc1ac87e"
           address="10.8.20.4"
           port="8112"
           apiport="8112"
           weight="32"/>
</nodes>
```
