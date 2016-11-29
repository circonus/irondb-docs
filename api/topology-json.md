Retrieving Topology JSON Data
=============================

This API call retrieves data for a given topology in JSON format.

Data will be returned as an array of JSON objects. The format of these objects is described below.

Description of JSON Objects
---------------------------

**URI:**   /topology/json/&lt;hash&gt;

**Method:**   GET

**Inputs:**

*hash* :   The hash of the topology for which to retrieve information.

**Outputs:**

*id* :   The UUID of the node.

*address* :   The IP Address of the node.

*port* :   The port on which the node is listening.

*apiport* :   The port on which the API is listening for the node.

*weight* :   A value representing how heavily the data to be stored on this node is weighted.

*n* :   The number of nodes on this ring on which data is stored.

Examples
--------

This example will use

```
/topology/json/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

*read* :   This is the command to read topology data from the server.

*json* :   This is the command to read data in JSON format.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef* :   This is the Topology Hash.

**Output:**

```
    [
       {"id":"1f846f26-0cfd-4df5-b4f1-e0930604e577","address":"10.8.20.1","port":8112,"apiport":8112,
    "weight":32,"n":2},
       {"id":"765ac4cc-1929-4642-9ef1-d194d08f9538","address":"10.8.20.2","port":8112,"apiport":8112,
    "weight":32,"n":2},
       {"id":"8c2fc7b8-c569-402d-a393-db433fb267aa","address":"10.8.20.3","port":8112,"apiport":8112,
    "weight":32,"n":2},
       {"id":"07fa2237-5744-4c28-a622-a99cfc1ac87e","address":"10.8.20.4","port":8112,"apiport":8112,
    "weight":32,"n":2}
    ]
```