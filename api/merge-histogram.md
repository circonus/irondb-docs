# Merging Histogram Data

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for merging histogram data into a Snowth node.

Raw binary data must be attached. The node that receives this will merge
this data into the local histogram LevelDB key/value store.

## Description

### URI

`/merge/histogram`

### Method

MERGE

### Headers

 * `X-Target-Topology: <topo_hash>` (required)
   * `topo_hash` The target topology into which data is merged.

## Examples

```
curl -X MERGE \
     -H 'X-Target-Topology: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef' \
     http://127.0.0.1:8112/merge/histogram
```
