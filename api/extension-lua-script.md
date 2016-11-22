Executing a Lua Extension
=========================

This API call will execute a loaded Lua extension and return the
results.

The available extensions can be found by making an API call to
"/extension/lua." Parameters to each extension are passed via a query
string.

Description of API call
-----------------------

`URI:`

:   /extension/lua/&lt;extension&gt;

`Method:`

:   GET

`Inputs:`

:   

    *extension*

    :   The extension to call. A list of available extensions can be
        found by making a call to "/extension/lua". The parameters for
        each extension there are passed via a query string.

`Output:`

:   The output will vary based on the lua extension called.

Examples
--------

This example uses

    /extension/lua/example_extension

In this example:

*extension*

:   This is the command to execute an extension.

*lua*

:   This is the command to execute a Lua extension.

*example\_extension*

:   This is the Extension Name.

`Output:`

    {"got_result":"true"}
