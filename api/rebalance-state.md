# Getting Topology Rebalance State

This API call is for viewing the current topology rebalance state.

Data will be returned as a JSON document. The fields in this document are described below.

## Description of JSON document

**URI:** /rebalance/state

**Method:** GET

**Output:**

*current* :   The current topology in which this node resides.

*next* :   The next topology for this node.

*state* :   Current rebalance state ({"TOPO_REBALANCE_IDLE", "TOPO_REBALANCE_VOTE",
    "TOPO_REBALANCE_REHASH", "TOPO_REBALANCE_REHASH_VOTE", "TOPO_REBALANCE_CLEANUP", "TOPO_REBALANCE_COMPLETE"})
