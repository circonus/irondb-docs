Loading a New Topology
======================

This API call will load a new topology onto an IRONdb node. It will not activate the topology; it will simply store and load it.

Description of API call
-----------------------

**URI:**   /topology/hash

**Method:**   POST

**Inputs:**

*hash* :   The hash for the new topology to load.

**Topology Data XML Format:**

*&lt;nodes&gt;* :   Top-Level XML for the topology.

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
/topology/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

*read* :   The command to handle topology data.

*Topology Hash:* 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

**POST Data:**

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