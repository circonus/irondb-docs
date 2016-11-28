# Retrieving Gossip JSON Data

This API call retrieves gossip information from a IRONdb node. Gossip data is information on how the nodes are communicating with each other and if any nodes are behind other nodes with regards to data replication.

Data will be returned as an array of JSON objects. The format of these objects is described below.

## Description of JSON objects

**URI:** :   /gossip/json

**Method:** :   GET

**Output:**  

*id* :   The UUID of the node of the gossip information.

 * *gossip\_time* :   The last time, in seconds, that this node received a gossip message.

    *gossip\_age*

    :   The difference, in seconds, between the last time this node
        received a gossip message and the current time.

    *topo\_current*

    :   The topology that is currently in use.

    *topo\_next*

    :   The "next" topology to use.

    *topo\_state*

    :   The state of the current topology.

    *latency*

    :   A JSON object that contains information on how far this node is
        lagging behind the other nodes. The entries will include the
        following:

    :   

        *uuid*

        :   The UUID of the node to which the current node is
            being compared.

        *latency\_seconds*

        :   The number of seconds that the current node is behind the
            specified node.

Examples
--------

The URI "/gossip/json" could yield the following example output:

    [
    {"id":"1f846f26-0cfd-4df5-b4f1-e0930604e577",
     "gossip_time":1409082055.744880,
     "gossip_age":0.000000,
     "topo_current":"0123456789abcdef0123456789abcdef0123456789abcdef01234
    56789abcdef",
     "topo_next":"-",
     "topo_state":"n/a",
     "latency": {
       "765ac4cc-1929-4642-9ef1-d194d08f9538":"0",
       "8c2fc7b8-c569-402d-a393-db433fb267aa":"0",
       "07fa2237-5744-4c28-a622-a99cfc1ac87e":"0"
      }
    },
    {"id":"765ac4cc-1929-4642-9ef1-d194d08f9538",
     "gossip_time":1409082055.744880,
     "gossip_age":0.000000,
     "topo_current":"0123456789abcdef0123456789abcdef0123456789abcdef01234
    56789abcdef",
     "topo_next":"-",
     "topo_state":"n/a",
     "latency": {
       "1f846f26-0cfd-4df5-b4f1-e0930604e577":"0",
       "8c2fc7b8-c569-402d-a393-db433fb267aa":"0",
       "07fa2237-5744-4c28-a622-a99cfc1ac87e":"0"
      }
    },
    {"id":"8c2fc7b8-c569-402d-a393-db433fb267aa",
     "gossip_time":1409082055.744880,
     "gossip_age":0.000000,
     "topo_current":"0123456789abcdef0123456789abcdef0123456789abcdef01234
    56789abcdef",
     "topo_next":"-",
     "topo_state":"n/a",
     "latency": {
       "765ac4cc-1929-4642-9ef1-d194d08f9538":"0",
       "1f846f26-0cfd-4df5-b4f1-e0930604e577":"0",
       "07fa2237-5744-4c28-a622-a99cfc1ac87e":"0"
      }
    },
    {"id":"07fa2237-5744-4c28-a622-a99cfc1ac87e",
     "gossip_time":1409082055.744880,
     "gossip_age":0.000000,
     "topo_current":"0123456789abcdef0123456789abcdef0123456789abcdef01234
    56789abcdef",
     "topo_next":"-",
     "topo_state":"n/a",
     "latency": {
       "765ac4cc-1929-4642-9ef1-d194d08f9538":"0",
       "8c2fc7b8-c569-402d-a393-db433fb267aa":"0",
       "1f846f26-0cfd-4df5-b4f1-e0930604e577":"0"
      }
    }
    ]
          
