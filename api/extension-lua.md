# Getting The List of Lua Extensions

This API call returns a list of all currently available Lua extensions on the
node.

Data will be returned as a JSON object. The fields in this document are
described below.

## Description

### URI

`/extension/lua`

### Method

GET

### Inputs

none

### Output

 * `<name>` : This key is the name of the upcoming extension. The value is another JSON object, defined as follows:
   * `params` : A JSON object containing the parameters for the extension. The JSON object is defined as follows:
     * `<param_name>` : This key is the name of the parameter. Each parameter will contain a JSON object defined as follows:
       * `name` : A description of the parameter.
       * `type` : The type of the parameter.
   * `description` : A text description of the Lua extension.

## Examples

```
curl http://127.0.0.1:8112/extension/lua
```

### Example 1 Output

```
{
  "example_ext": {
    "params": {
      "sample_param": {
        "name": "sample_param",
        "type": "integer"
      },
      "sample_param2": {
        "name": "sample_param2",
        "type": "string"
      }
    },
    "description": "A sample extension"
  },
  "example_ext2": {
    "params": {
      "sample_param": {
        "name": "sample_param",
        "type": "integer"
      },
      "sample_param2": {
        "name": "sample_param2",
        "type": "string"
      }
    },
    "description": "Another sample extension"
  }
}
```
