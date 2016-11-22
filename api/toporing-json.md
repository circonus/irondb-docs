Retrieving Toporing JSON Data
=============================

This API call retrieves toporing data for a given topology in JSON
format.

Data will be returned as an array of JSON objects. The format of these
objects is given below.

Description of JSON objects
---------------------------

`URI:`

:   /toporing/json/&lt;hash&gt;

`Method:`

:   GET

`Inputs:`

:   

    *hash*

    :   The hash of the topology to retrieve information for.

`Outputs:`

:   

    *id*

    :   The UUID of the node.

    *idx*

    :   The index of an entry in the toporing. This is a number between
        1 and n, where "n" is the weight of the node.

    *location*

    :   The given location of the node.

Examples
--------

This example retrieves a simplified topology for a 3-node cluster,
assuming a weight of 3.

This example uses

    /toporing/json/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

In this example:

*toporing*

:   This is the command to read toporing data from the server.

*json*

:   This is the command to read data in JSON format.

*0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef*

:   This is the Topology Hash.

`Output:`

    [
       {"id":"1f846f26-0cfd-4df5-b4f1-e0930604e577","idx":1,"location":11.000000},
       {"id":"1f846f26-0cfd-4df5-b4f1-e0930604e577","idx":2,"location":22.000000},
       {"id":"1f846f26-0cfd-4df5-b4f1-e0930604e577","idx":3,"location":33.000000},
       {"id":"765ac4cc-1929-4642-9ef1-d194d08f9538","idx":1,"location":44.000000},
       {"id":"765ac4cc-1929-4642-9ef1-d194d08f9538","idx":2,"location":55.000000},
       {"id":"765ac4cc-1929-4642-9ef1-d194d08f9538","idx":3,"location":66.000000},
       {"id":"8c2fc7b8-c569-402d-a393-db433fb267aa","idx":1,"location":77.000000},
       {"id":"8c2fc7b8-c569-402d-a393-db433fb267aa","idx":2,"location":88.000000},
       {"id":"8c2fc7b8-c569-402d-a393-db433fb267aa","idx":3,"location":99.000000}
    ]
          
