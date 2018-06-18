# Retrieving Gossip XML Data

This API call retrieves gossip information from a Snowth node. Gossip data is
information on how the nodes are communicating with each other and if any nodes
are behind other nodes with regards to data replication.

Data will be returned an XML object. The format of this object is described
below.

## Description

### URI

`/gossip/xml`

### Method

GET

### Output


 * `<nodes>` : The top-level element for the topology.
   * `<node>` : The container for all the information for a single node in the cluster. There will x of these elements, where "x" is the number of nodes in the cluster.
     * Attributes:
       * `id` : The UUID of the node whose gossip information follows.
       * `gossip_time` : The last time, in seconds, that this node received a gossip message.
       * `gossip_age` :  The difference, in seconds, between the last time this node received a gossip message and the current time.
       * `topo_current` : The topology that is currently in use.
       * `topo_next` : The "next" topology to use.
       * `topo_state` : The state of the current topology.
     * `<latency>` : The element containing latency information for all non-local nodes. 
       * `<node>` : The element containing latency information for a non-local node.
         * Attributes:
           * `id` : The UUID of the node to which the current node is being compared.
           * `diff` : The number of seconds that the current node is behind the specified node.

## Examples

```
curl http://127.0.0.1:8112/gossip/xml
```

### Example 1 Output

```xml
<nodes>
  <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577"
        gossip_time="1409082055.744880"
        gossip_age="0.000000"
        topo_current="0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
        topo_next="-"
        topo_state="unknown">
    <latency>
      <node id="765ac4cc-1929-4642-9ef1-d194d08f9538" diff="0"/>
      <node id="8c2fc7b8-c569-402d-a393-db433fb267aa" diff="0"/>
      <node id="07fa2237-5744-4c28-a622-a99cfc1ac87e" diff="0"/>
    </latency>
  </node>
  <node id="765ac4cc-1929-4642-9ef1-d194d08f9538"
        gossip_time="1409082055.744880" 
        gossip_age="0.000000" 
        topo_current="0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef" 
        topo_next="-" 
        topo_state="unknown">
    <latency>
      <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577" diff="0"/>
      <node id="8c2fc7b8-c569-402d-a393-db433fb267aa" diff="0"/>
      <node id="07fa2237-5744-4c28-a622-a99cfc1ac87e" diff="0"/>
    </latency>
  </node>
  <node id="8c2fc7b8-c569-402d-a393-db433fb267aa"
        gossip_time="1409082055.744880"
        gossip_age="0.000000"
        topo_current="0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
        topo_next="-"
        topo_state="unknown">
    <latency>
      <node id="765ac4cc-1929-4642-9ef1-d194d08f9538" diff="0"/>
      <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577" diff="0"/>
      <node id="07fa2237-5744-4c28-a622-a99cfc1ac87e" diff="0"/>
    </latency>
  </node>
  <node id="07fa2237-5744-4c28-a622-a99cfc1ac87e"
        gossip_time="1409082055.744880"
        gossip_age="0.000000"
        topo_current="0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
        topo_next="-"
        topo_state="unknown">
    <latency>
      <node id="765ac4cc-1929-4642-9ef1-d194d08f9538" diff="0"/>
      <node id="8c2fc7b8-c569-402d-a393-db433fb267aa" diff="0"/>
      <node id="1f846f26-0cfd-4df5-b4f1-e0930604e577" diff="0"/>
    </latency>
  </node>
</nodes>       
```
