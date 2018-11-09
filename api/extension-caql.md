# Issuing CAQL Queries

The Circonus Analytics Query Language (CAQL) allows the user to issue complex queries against metric data residing in IRONdb.
The [CAQL Reference](https://login.circonus.com/resources/docs/user/caql_reference.html) provides comprehensive documentation about functionality offered by the language.


## Description

### URI

`/extension/lua/caql_v1`

### Method

GET

### Inputs

Parameters can be submitted as url-encoded query string parameters or as JSON payload.

The following parameters are supported:

* `query` : The query to execute.

* `start` : The start time from which to pull data, represented in seconds since the epoch. This value is inclusive (data for the given start time will be pulled).

* `end` : The end time up to which data is pulled, represented in seconds since the epoch. This value is exclusive (data up to, but not including, the given end time will be pulled).

* `period` : The period, in seconds, for which to get data rollups.

* `_timeout` : Specify a timeout for CAQL processing in seconds. Optional. Default = 4.5.

* `preview` : Boolean parameter ("true"/"false") that enables CAQL preview mode. In preview mode caql processing will be faster but results will only be approximated. Optional. Default = false.

## Examples

```
echo '{
  "query":"12+3",
  "start":1474275000,
  "end":1474275240,
  "period":60
}' | curl http://127.0.0.1:8112/extension/lua/caql_v1 --data @-
```

Equivalently we can pass the paraemeters via the query string:


```
curl  'http://127.0.0.1:8112/extension/lua/caql_v1?start=1474275000&end=1474275240&period=60&query=12+3'
```

### Output

```json
[
 [1474275000,[15]],
 [1474275060,[15]],
 [1474275120,[15]],
 [1474275180,[15]]
]
```
