Retrieving Topology XML Data
============================

This API call retrieves data for a given topology in XML format.

Data will be returned as an XML document. The format of this document is described below.

Description of XML document
---------------------------

**URI:** :   /topology/xml/&lt;hash&gt;

**Method:** :   GET

**Inputs:**  

*hash* :   The hash of the topology for which to retrieve information.

**Outputs:**

*&lt;nodes&gt;* :   The Top-Level XML for the topology.

* *Attributes*

 * *n* :   The number of nodes on which the data will be stored.

* *Elements*

 * *&lt;node&gt;* :   The container for all the information on a single node in the cluster. There will be up to x of these, where "x" is the number of nodes in the cluster.

   * *Attributes*

     * *id* :   The UUID of the node.

     * *address* :   The IP Address of the node.

     * *port* :   The port on which the node is listening.

      * *apiport* :   The port on which the API is listening for the node.

      * *weight* :   A value representing how heavily the data to be stored on this node is weighted.

Examples
--------

This example will use

```
/topology/xml/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

*topology* :   This is the command to read topology data from the server.

*xml* :   This is the command to read data in XML format.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef* :   This is the Topology Hash.

**Output:**

```
    <nodes n="2">
      <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
               address="10.8.20.1"
               port="8112"
               apiport="8112"
               weight="32"/>
      <node id="765ac4cc-1929-4642-9ef1-d194d08f9538"
               address="10.8.20.2"
               port="8112"
               apiport="8112"
               weight="32"/>
      <node id="8c2fc7b8-c569-402d-a393-db433fb267aa"
               address="10.8.20.3"
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