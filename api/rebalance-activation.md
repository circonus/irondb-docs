Activating A New Topology Rebalance
=========================

This API call is for rebalancing to a new topology.

Description of API call
-----------------------

**URI:**   /rebalance/activate/&lt;hash&gt;

**Method:**  POST 

**Inputs:**

*hash* :   The hash of the new topology after the rebalance.

Examples
--------

This example will use

```
/rebalance/activate/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

*activate* :   This is the command to activate a new topology rebalance.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef* :   This is the hash for the transition.
