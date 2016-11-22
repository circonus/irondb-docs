Merging Histogram Data
======================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for merging histogram data into a Snowth node.

Raw binary data must be attached. The node that receives this will merge
this data into the local histogram LevelDB key/value store.

Description of API call
-----------------------

`URI:`

:   /merge/histogram

`Method:`

:   MERGE

`Headers:`

:   One header is required for this call. It is described below.

:   

    *X-Target-Topology*

    :   The target topology into which data is merged.

Examples
--------

    /merge/histogram

    Header: "X-Target-Topology: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
