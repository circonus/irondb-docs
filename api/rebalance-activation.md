# Activating A New Topology Rebalance

This API call is for rebalancing to a new topology.

## Description

### URI

`/rebalance/activate/<hash>`

### Method

POST

### Inputs

 * `hash` : The hash of the new topology after the rebalance.

## Examples

```
curl -X POST \
  http://127.0.0.1:8112/rebalance/activate/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

 * `activate` : This is the command to activate a new topology rebalance.
 * `0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef` : This is the hash for the transition.
