Loading a New Lua Extension
===========================

This API call will load a new Lua extension into IRONdb.

Description of API call
-----------------------

**URI:**   /load/lua

**Method:**   POST

**Headers:**   Two headers are required for this POST call. They are described below.

*X-Script-Name* :   The name of the script to load.

*X-Script-Timestamp* :   A timestamp (represented in seconds since the epoch) for when the message was sent to the IRONdb node.

Examples
--------

```
    /load/lua

    Header: "X-Script-Name: test"
    Header: "X-Script-Timestamp: 1380000000
    Contents: "print 'hello, world!\n'"
```