Writing Text Data
=================

This API call is for writing text data into the Snowth cluster. It sends a JSON object containing the data to be added to the cluster.

Description of JSON object
--------------------------

**URI:**   /write/text

**Method:**   PUT | POST

**JSON Format:**   

*metric* :   The name of the metric for which data is added.

*id* :   The UUID of the check for the metric for which data is added.

*offset* :   The timestamp, represented in time since the epoch, for which data added.

*value* :   The text string to add to the Snowth cluster.

This example uses

```
/write/text
```

The example JSON object below will add data to the IRONdb cluster for two text metrics, named "textexample1" and "textexample2". The data will be added at offset 1408724400 (August 22, 2014, 12:20:00 GMT).

**Attached Text:**

```
    [
       { "offset": "1408724400", "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e", "metric": "textexample1", "value": "this_is_a_test"},

       { "offset": "1408724400", "id": "ae0f7f90-2a6b-481c-9cf5-21a31837020e", "metric": "textexample2", "value": "this_is_also_a_test"},
    ]
```