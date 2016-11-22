Retrieving Raw Histogram Data
=============================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is for retrieving raw binary data for a histogram metric.
It will return data for every metric that belongs on the node specified
in the nodeid field.

Description of API call
-----------------------

`URI:`

:   /histogram/raw/&lt;nodeid&gt;

`Method:`

:   GET

`Inputs:`

:   

    *nodeid*

    :   The UUID of the node that is requesting data.

`Query String Values:`

:   

    *hex\_hash*

    :   An optional hex value (00-FF) that will pull back only metrics
        that an internal hashing algorithm matches to the given hash.
        All 256 hex hash values combined will represent all data.
        Leaving this value out will return all data that belongs on the
        given nodeid.

This example will pull all histogram data that belongs on a node in a
topology with the ID "53276c66-3bb0-47fc-8474-cb2045d3da72".

    /histogram/raw/53276c66-3bb0-47fc-8474-cb2045d3da72

In this example:

*histogram*

:   This is the command to read histogram data from the server.

*raw*

:   This is the command to read raw data from the server.

*53276c66-3bb0-47fc-8474-cb2045d3da72*

:   This is the Node ID.

`Output:`

The output will be raw, binary output that represents all the histogram
data on the target node that should also be stored on the node with ID
53276c66-3bb0-47fc-8474-cb2045d3da72.
