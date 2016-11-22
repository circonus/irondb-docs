Getting List of Lua Extensions
==============================

This API call returns a list of all currently available Lua extensions
on the node.

Data will be returned as a JSON object. The fields in this document are
described below.

Description of JSON object
--------------------------

`URI:`

:   /extension/lua

`Method:`

:   GET

`Output:`

:   

    *&lt;name&gt;*

    :   This key has a variable name. It will be the name of the
        upcoming extension. The value is another JSON object, defined as
        follows:

    :   

        *params*

        :   A JSON object containing the parameters for the extension.
            The JSON object is defined as follows:

        :   

            *&lt;param\_name&gt;*

            :   This key has a variable name. It will be the name of
                the parameter. Each parameter will contain a JSON object
                defining it as follows:

            :   

                *name*

                :   A description of the parameter.

                *type*

                :   The type of the parameter.

        *description*

        :   A text description of the Lua extension.

Examples
--------

This example uses

    /extension/lua

`Output:`

    {"example_ext":{"params":{"sample_param":{"name":"sample_param","type":"integer"},"sample_param2:{"name":"sample_param2","type":"string"}},"description":"A sample extension"},

    "example_ext2":{"params":{"sample_param":{"name":"sample_param","type":"integer"},"sample_param2:{"name":"sample_param2","type":"string"}},"description":"Another sample extension"}}
          
