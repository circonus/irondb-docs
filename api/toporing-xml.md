Retrieving Toporing XML Data
============================

This API call retrieves toporing data for a given topology in XML format.

Data will be returned as a XML object. The format of this object is given below.

Description of XML object
-------------------------

**URI:**   /toporing/xml/&lt;hash&gt;

**Method:**   GET

**Inputs:**

*hash* :   The hash of the topology for which to retrieve information.

**Outputs:**

*&lt;vnodes&gt;* :   The Top-Level XML for the topology.

* *Attributes*

 * *n* :   The number of nodes on which the data will be stored.

* *Elements*

 * *&lt;vnode&gt;* :   The container for all the information for a single virtual node in the cluster.

   * *Attributes*

     * *id* :   The UUID of the node.

     * *idx* :   The index of an entry in the toporing. This is a number between 1 and n, where "n" is the weight of the node.

     * *location* :   The given location of the node.

Examples
--------

This example retrieves a simplified topology for a 3-node cluster, assuming a weight of 3.

This example uses

```
/toporing/json/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

*toporing* :   This is the command to read toporing data from the server.

*xml* :   This is the command to read data in XML format.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef* :   This is the Topology Hash.

**Output:**

```
    <vnodes n="2">
      <vnode id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
                idx="1"
                location="11.000000"/>
      <vnode id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
                idx="2"
                location="22.000000"/>
      <vnode id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
                idx="3"
                location="33.000000"/>
      <vnode id="765ac4cc-1929-4642-9ef1-d194d08f9538"
                idx="1"
                location="44.000000"/>
      <vnode id="765ac4cc-1929-4642-9ef1-d194d08f9538"
                idx="2"
                location="55.000000"/>
      <vnode id="765ac4cc-1929-4642-9ef1-d194d08f9538"
                idx="3"
                location="66.000000"/>
      <vnode id="8c2fc7b8-c569-402d-a393-db433fb267aa"
                idx="1"
                location="77.000000"/>
      <vnode id="8c2fc7b8-c569-402d-a393-db433fb267aa"
                idx="2"
                location="88.000000"/>
      <vnode id="8c2fc7b8-c569-402d-a393-db433fb267aa"
                idx="3"
                location="99.000000"/>
    </nodes>
```