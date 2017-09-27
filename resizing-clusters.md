# Resizing Clusters

The essential steps to changing the topology of an existing IRONdb cluster are
as follows:
1. Create your new topology.
1. Load the new topology to all nodes that will be part of the new cluster.
1. Start the rebalance operation on each node, which starts the migration to
the new topology.

These procedures utilize the IRONdb [REST API](api.md) which, by default,
listens on TCP port 8112. See the [Rebalancing APIs
reference](rebalance-apis.md) for details on the endpoints mentioned below.

## Adding Nodes

An existing IRONdb cluster has two nodes with write factor of 2. A new node is
prepared by running the [installation](installation.md) which creates a
standalone node with its own topology. We want to combine these three nodes
together to create a three-node cluster with write factor of 3.

* Combine the XML content of both old topologies to [create a new topology XML
  document](installation.md#create-topology-layout) with three nodes.

* [Import](installation.md#import-topology) the new topology to get the new
  topology hash. Unlike an initial cluster setup, this file does not need to be
  placed anywhere special, as we are going to load this via the REST API.

  ```/opt/circonus/bin/snowthimport -f newtopology.xml -t .``` 
  
  This will generate a canonical topology configuration file named for the new
  topology hash. Note this hash string for use in later steps.

* Load the new topology to each of the three nodes via POST to `/topology/[new topology
  hash]` with new topology config: 
  
  ```curl -X POST -d@<hash> http://<node>:<api-port>/topology/<hash>```

  This will install the new topology config in the existing topology directory,
  alongside the current topology for that node.
  
* Start migration to the new topology on each of the three nodes via POST to
  `/rebalance/activate/[new topology hash]`:
  
  ```curl -X POST http://<node>:<api-port>/rebalance/activate/<hash>``` 
  
  IRONdb will start rebalancing the data in the old topology to the new
  topology once all the nodes in old and new topology receive the rebalance
  activation POST with the same new topology hash. Rebalancing involves
  recalculating the node ownership for each individual metric stream, and then
  sending that stream to the new node. All metric data remains available during
  a rebalance, under the old topology. New, incoming metric data is
  replicated to _both_ old and new topologies.

  After all nodes complete the rebalance, they will switch their active topology
  from old to new. Each node will then kick off a delete operation of any
  metrics that no longer belong on that node.

* To view progress, retrieve the rebalance state via GET of `/rebalance/state`:
  
  ```curl http://<node>:<api-port>/rebalance/state```
  
* To abort the rebalance, POST to `/rebalance/deactivate/[new topology hash]`
  
  ```curl -X POST http://node:<api-port>/rebalance/deactivate/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3```

## Removing Nodes

Shrinking a cluster is basically the same as adding, above:
* Create a new topology with the nodes that should remain.
* Load the new topology to the nodes that should remain.
* Start rebalance to new topology on the nodes that should remain.

One difference is that nodes that are not in the new topology do not
automatically clean up their data, which avoids delaying the cleanup phase
unnecessarily. It is up to the operator to do this as part of decommissioning
the unused nodes.
