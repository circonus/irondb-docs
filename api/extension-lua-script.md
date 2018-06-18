# Executing a Lua Extension

This API call will execute a loaded Lua extension and return the results.

Refer to [Getting The List of Lua Extensions](/api/extension-lua.md) for
instructions on finding the list of available extensions.

## Description

### URI

`/extension/lua/<extension>`

### Method

GET

### Inputs

 * `extension` : The extension to call. Any parameters for the extension may be
   passed via a query string.

### Output

The output will vary based on the Lua extension called.

## Examples

```
curl http://127.0.0.1:8112/extension/lua/example_extension
```

In this example:

 * `extension` : This is the command to execute an extension.
 * `lua` : Indicates a Lua extension.
 * `example_extension` : Extension name.

### Example 1 Output

```
{"got_result":"true"}
```
