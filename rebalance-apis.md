# Rebalance APIs and procedure 

Procedure

For example, we have two IRONdb clusters. First cluster has two nodes with write factor of 2. Second cluster has single node with write factor of 1.
We want to combine these three nodes together to create a three node cluster with write factor of 3.

- Combine first cluster topology and second cluster topology xml and create new topology xml with three nodes
- Snowth import the new topology and create new topology hash uuid
  snowthimport -f topology.xml -t . 
  it will generate a topology uuid hash file for the input topology.xml
- Load the new topology to all three nodes via POST /topology/[new topology uuid] with new topology xml 
  c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3 is the new topology xml with topology hash code as filename  
  curl -X POST -d@c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3 http://node:8112/topology/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3
- Start rebalance to new topology to all three nodes via POST /rebalance/activate/[new topology uuid]
  curl -X POST http:/node:8112/rebalance/activate/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3 
  IRONdb will start rebalance the data in the old topology to new topology once all the nodes in old and new topology receive the rebalance activation POST with the same new topology uuid.     
  After all nodes complete the rehash and rebalance the text/raw/nnt/hist, it will switch the topology from old topology to new topology in all nodes and cleanup the data for nodes in old topology.
  During the rebalance, the inflight inject data will be replicated to both old and new topology.
- Retrieve the rebalance state via GET /rebalance/state
  curl http://node:8112/rebalance/state
- Abort the rebalance via POST /rebalance/deactivate/[new topology uuid]
  curl -X POST http://node:8112/rebalance/deactivate/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3

For example, if you want to remove one nodes from the three nodes cluster created above
- Create a new two node cluster topology
- Loading the new two node topology to all two nodes via POST /topology/[new topology uuid] with new topology xml
- Start Rebalance to new topology to all two nodes via POST /rebalance/activate/[new topology uuid]

---

 * [Loading a New Topology](api/topology.md)

 * [Starting Rebalance to new Topology](api/rebalance-activation.md)

 * [Retrieving Rebalance state](api/rebalance-state.md)

 * [Aborting Rebalance to new Topology](api/rebalance-deactivation.md)
