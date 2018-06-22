# Retrieving Raw Data

Fetches raw (full resolution) numeric data from the [raw
database](/configuration.md#rawdatabase).  Data is returned as an array of
tuples of `[timestamp in milliseconds, value]`

## Description

### URI

`/raw/<uuid>/<metric>?start_ts=<start>&end_ts=<end>`

### Method

GET

### Inputs

 * `metric`   : The name of the metric for which to pull data (url encoded).
 * `uuid`     : The UUID of the check to which the metric belongs.
 * `start_ts` : The start time from which to pull data, represented in seconds.milliseconds since the unix epoch.
 * `end_ts`   : The end time up to which data is pulled, represented in seconds.milliseconds since the unix epoch.
  
> A note on the `start_ts` and `end_ts` parameters:

The format is `<seconds since epoch>.<milliseconds>`.  In terms of printf, it
is `%lu.%03lu`, i.e., `1516820826.120`.  **While this might look like a
float,** it is, in fact, a strict textual format that requires exactly three
digits after the decimal point. These must always be included, even if they are
`000`.

For example, 12345.6 does **not** mean 12345 seconds and 600 milliseconds; it
is an illegal format.  The `<milliseconds>` portion of the timestamp must
always be 3 digits to represent values from 000 to 999.

## Examples

```
curl http://127.0.0.1:8112/raw/fc85e0ab-f568-45e6-86ee-d7443be8277d/online?start_ts=1529509020.000&end_ts=1529509200.000
```

In this example:

 * `raw`     : The raw command
 * `fc85e0ab-f568-45e6-86ee-d7443be8277d` : The UUID of the check
 * `online`  : The name of the metric to retrieve data for
 * `1529509020.000` : 2018-06-20T15:37:00+00:00 and zero milliseconds
 * `1529509200.000` : 2018-06-20T15:40:00+00:00 and zero milliseconds

### Example 1 Output

```json
[[1529509063064,0],[1529509122985,0],[1529509183764,0]]
```

Note that the returned timestamps are **milliseconds** since unix epoch and
represent the timestamp on each incoming data row.

