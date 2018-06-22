# Activating A New Topology

Switches a node to a new topology.

> **Caution**
>
> THIS IS A DANGEROUS CALL. DO NOT USE IT UNLESS YOU ARE SURE YOU KNOW WHAT YOU ARE DOING.

## Description

### URI
`/activate/<hash>`

### Method
GET

### Inputs

 * `hash` : The hash of the new topology after the transition.

## Examples

```
curl http://127.0.0.1:8112/activate/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```

In this example:

 * `activate` : This is the command to activate a new topology.
 * `0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef` : The new topology's hash 
