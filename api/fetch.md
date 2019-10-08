# Retrieving and Tranforming Data

The /fetch API provides fast, one-request access to common complex data extraction requirements.
It allows for fetch submissions in both FlatBuffers and JSON formats, and returns DF4 output format available in both FlatBuffers and JSON encoding.

## Description of API

### URI

`/fetch`

### Method

POST

### Inputs

 * Requires the `X-Circonus-Account` header.
 * A document describing the streams to fetch, how to transform them, and reduce them into a result set.

#### Fetch JSON

```
{ "start": <double epoch seconds>,
  "period": <double seconds>,
  "count": <positive integer>,
  "streams": [
    { ... stream defintion ... },
    ...
  ]
  "reduce": [
    { ... reduce defition ... },
    ...
  ]
}
```

A stream definition form:
```
{ uuid: <uuid>,
  name: <canonical name of metric>,
  kind: <numeric|histogram|text>,
  transform: <transform>,
  transform_params: [ string parameter list ]
}
```

A reduce definition form:
```
{ "label": <string label>,
  "method": <reduce method>,
  "method_params": [ string parameter list ]
}
```

### Transforms

#### Numeric (`kind` = `numeric`)

 * `count` - the number of measurements in the period.
 * `average` - the average of measurements in the period.
 * `stddev` - the standard deviation of measurements in the period.
 * `derivative` - the rate of change of the measurements in the period.
 * `derivative_stddev` - the standard deviation of the rate of change of the measurements in the period.
 * `counter` - the positive rate of change of the measurements in the period.
 * `counter_stddev` - the standard deviation of the positive rate of change of the measurements in the period.
 
#### Histogram (`kind` = `histogram`)

 * `none` - pass the input through unmodified.

#### Text (`kind` = `text`)

 * `none` - pass the input through unmodified.

### Reductions

#### `pass` - pass the inputs to outputs unmodified.

 * `method_params` none
 * Inputs can be numeric, histogram, or text.

#### `groupby_mean` - group inputs and calculate a mean over the grouping

 * `method_params` a list of tag categories on which to perform grouping
 * Inputs must be numeric.

#### `groupby_sum` - group inputs and calculate a sum over the grouping

 * `method_params` a list of tag categories on which to perform grouping
 * Inputs must be numeric.

#### `groupby_merge` - group inputs and merge into a histogram stream.

 * `method_params` a list of tag categories on which to perform grouping
 * Inputs must be either numeric or histogram.

#### `mean` - calculate the mean across input streams.

 * `method_params` none
 * Inputs must be numeric.
 
#### `merge` - group inputs and merge into a histogram stream.

 * `method_params` none
 * Inputs must be either numeric or histogram.
 
#### `sum` - calculate the sum across input streams.

 * `method_params` none
 * Inputs must be numeric.
 
#### `topk` - filter a set of inputs to the top K

 * `method_params` : `[ K, <mech>, <mech_param> ]`
 * Inputs must be either numeric or histogram.

    Allowable `mech` values are `mean` (default), `max`, or `quantile`.  The `quantile` `mech value requires a `mech_param` in the range [0,1].  All measurements in the input stream are accumulated and a mean or quantile is calculated.  This calculated value is used as a rank for the stream and the K largest ranks are selected and passed to the output set.
