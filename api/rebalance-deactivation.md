Abort The Current In Progress Topology Rebalance
=========================

This API call is for aborting the current rebalancing to a new topology.

Description of API call
-----------------------

**URI:**   /rebalance/deactivate/&lt;hash&gt;

**Method:**  POST 

**Inputs:**

*hash* :   The hash of the new topology after the rebalance.

Examples
--------

This example will use

```
/rebalance/deactivate/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

*deactivate* :   This is the command to activate a new topology rebalance.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef* :   This is the hash for the transition.
