Posting JLOG Data Internally
============================

> **Warning**
>
> THIS API CALL IS FOR INTERNAL USE ONLY

This API call is used by Snowth nodes to pass data back and forth.

Data will be sent in a binary format using JLOG. The client node will
read the binary messages, decode them, and process the data as needed.

The types of messages that can be attached are as follows:

-   NNT Data Messages
-   NNT Delete Messages
-   Text Data Messages
-   Text Delete Messages
-   Histogram Data Messages
-   Histogram Delete Messages
-   Log Messages
-   Load Lua Commands

Description of API call
-----------------------

`URI:`

:   /journal/&lt;count&gt;

`Method:`

:   POST

`Inputs:`

:   

    *count*

    :   The number of messages attached to be processed.

This example uses

    /journal/10

In this example:

*journal*

:   This is the command to read journal data.

*10*

:   This is the Number of Messages included.


