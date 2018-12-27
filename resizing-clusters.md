# Resizing Clusters

The essential steps to changing the topology of an existing IRONdb cluster are
as follows:
* Create your new topology.
* Load the new topology to all nodes that will be part of the new cluster.
* Start the "rebalance" operation on each node, which begins the migration of
  metric data to the new topology.

Rebalancing involves recalculating the node ownership for each individual
metric stream, and then sending that stream to the new owning node. All metric
data remain available during a rebalance, under the old topology. New,
incoming metric data is replicated to _both_ old and new topologies.

After all nodes complete the rebalance, they will switch their active topology
from old to new. Each node will then kick off a delete operation of any
metrics that no longer belong on that node.

A helper tool exists to simplify the procedure, and its use is illustrated
below. Both additions and removals may be performed in the same operation,
subject to the restrictions stated in the [Caveats](#caveats) section below.

The helper tool utilizes the IRONdb [REST API](api.md) which, by default,
listens on TCP port 8112. See the [Rebalancing APIs
reference](rebalance-apis.md) for details. The helper tool is not necessary in
order to perform a resize; the same operation may be performed using the APIs
directly.

## Caveats

Rebalance cannot be used to transform a cluster with no sides into a sided
cluster, or vice versa. Such a change requires [migrating to a new
cluster](/migrating-clusters.md).

When removing nodes from a cluster, no more than `W-1` (one less than the
number of write copies) nodes may be removed in a rebalance operation. For
example, a cluster with `W=3` may have a maximum of 2 nodes removed at a time.

If resizing a sided cluster, the new cluster topology must still have at least
`W/2` (half the number of write copies) nodes on each side, to ensure that the
promise of metric distribution across sides can be maintained. For example, a
sided cluster with `W=3` must still have at least 2 nodes on each side in the
new topology (fractional values are rounded up to the nearest integer.)

## Adding Nodes

An existing IRONdb cluster has two nodes with write factor of 2. A new node is
prepared by running the [installation](installation.md) which creates a
standalone node with its own topology. We want to combine these three nodes
together to create a three-node cluster, maintaining 2 write copies.

We will use the cluster resizing tool, `/opt/circonus/bin/resize_cluster`. Run
this with the `-h` option for details on the available options.

* Choose one of the existing cluster nodes and note its IP address and API
  port. This will be the "bootstrap node" from which the resize tool will fetch
  the existing cluster's topology. If you do not specify the API port, the
  default (8112) will be assumed.

* Note the new node's IP address and node UUID, and, if the cluster is sided,
  whether the node will be added to side "a" or "b".

* Run the resize tool, specifying the new node with a comma-separated tuple of
  IP address, node ID, and optionally a side. If adding more than one node,
  specify the `-a` option multiple times.

  ```/opt/circonus/bin/resize_cluster -b <bootstrap_node_ip[:port]> -a <new_ip,new_uuid>```

* A summary of the new topology will be displayed, along with a listing of the
  existing cluster and the proposed changes. Unless you specified the `-y`
  (always answer "yes") option, you will be asked to confirm the changes before
  any actual work begins.

* Once the changes are confirmed, IRONdb will start rebalancing the data.
  The new topology hash will be shown once it has been calculated.

  After all nodes complete the rebalance, they will switch their active topology
  from old to new. Each node will then kick off a delete operation of any
  metrics that no longer belong on that node.

* To view progress, retrieve the [rebalance state](/api/rebalance-state.md) via
  GET of `/rebalance/state`:

  ```curl http://<node>:<api-port>/rebalance/state```

* To abort the rebalance, POST to `/rebalance/deactivate/[new topology hash]`

  ```curl -X POST http://<node>:<api-port>/rebalance/deactivate/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3```

  on every node, including any new nodes that were added.

## Removing Nodes

Shrinking a cluster is basically the same as adding, above:
* Create a new topology with the nodes that should remain.
* Load the new topology to the nodes that should remain.
* Start rebalance to new topology on the nodes that should remain.

One difference is that nodes that are not in the new topology do not
automatically clean up their data, which avoids delaying the cleanup phase
unnecessarily. It is up to the operator to do this as part of decommissioning
the unused nodes.

We will use the cluster resizing tool, `/opt/circonus/bin/resize_cluster`. Run
this with the `-h` option for details on the available options.

* Choose a node that will be staying in the cluster and note its IP address and
  API port. This will be the "bootstrap node" from which the resize tool will
  fetch the existing cluster's topology. If you do not specify the API port,
  the default (8112) will be assumed.

* Note the node UUID of the node(s) that will be removed.

* Run the resize tool, specifying the removed nodes by their node UUID. If
  removing more than one node, specify the `-r` option multiple times.

  ```/opt/circonus/bin/resize_cluster -b <bootstrap_node_ip[:port]> -r <removed_uuid>```

* A summary of the new topology will be displayed, along with a listing of the
  existing cluster and the proposed changes. Unless you specified the `-y`
  (always answer "yes") option, you will be asked to confirm the changes before
  any actual work begins.

* Once you have confirmed the changes, IRONdb will start rebalancing the data.
  The new topology hash will be shown once it has been calculated.

* To view progress, retrieve the [rebalance state](/api/rebalance-state.md) via
  GET of `/rebalance/state`:

  ```curl http://<node>:<api-port>/rebalance/state```

* To abort the rebalance, POST to `/rebalance/deactivate/[new topology hash]`

  ```curl -X POST http://<node>:<api-port>/rebalance/deactivate/c790e663badf603ac555e02a07bf195f06932f271ee1f2d7fae104b3a1b232f3```

  on every node that is part of the new topology.
