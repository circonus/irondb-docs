# Writing Raw Data

In contrast to the other submission APIs ([numeric](./write-nnt.md),
[text](./write-text.md), [histogram](./write-histogram.md)), which accept
specifically-typed data, the raw API accepts direct input of measurement data
at arbitrary frequencies. It stores every measurement as it was received, for a
configurable amount of time, before aging it out to a rollup format.

Metric records are in one of several formats, and are accepted as either
tab-separated values or as FlatBuffer messages.

## Metric Record Formats
Raw metric records may be submitted in one of several formats, depending on the
type of metric data contained within.

### TSV Format

**URI:** /raw

**Method:** PUT | POST

**Format:** Multiple (see below)

#### TSV Numeric Or Text Metrics

Individual numeric or text metrics submitted to the raw endpoint as lines of
ASCII characters use the following format, referred to as an `M` record:
```
M TIMESTAMP UUID NAME TYPE VALUE
```

Components are separated by TAB characters. Multiple records may be sent in the
same operation, separated by newline (`\n`).

*M* : Denotes an `M` record.

*TIMESTAMP* : An epoch timestamp recording the time of the observation, with
milliseconds. In terms of format, it is `%lu.%03lu`, i.e., `1516820826.120`.
**While this might look like a float,** it is, in fact, a strict textual format
that requires exactly three digits after the decimal point. These must always
be included, even if they are `000`.

*UUID* : An identifier of the account and check to which this metric belongs.
Despite its name, this identifier must be in the form:
```
TARGET`MODULE`CIRCONUS_NAME`lower-cased-uuid
```
  * `TARGET` is conventionally the IP address of the check target, but may be
    any meaningful string identifying the subject of the check.
  * `MODULE` is conventionally the name of the [Reconnoiter check module](https://github.com/circonus-labs/reconnoiter/tree/master/src/modules).
  * `CIRCONUS_NAME` is what determines both the account and check to which this
    metric belongs. It has the form `c_ACCOUNT-ID_CHECK-BUNDLE-ID::MODULE`.
    `ACCOUNT-ID` is the most significant, as this is how metric data is partitioned
    within IRONdb.
  * `lower-cased-uuid` is the check UUID, lower-cased.

*NAME* : The name of this metric. The conventional format for hierarchical
naming is "name\`subname\`subsubname\`etc."

*TYPE* : The type of data that the `VALUE` represents:
  * `i`: int32
  * `I`: uint32
  * `l`: int64
  * `L`: uint64
  * `n`: double
  * `s`: string

*VALUE* : The value observed. `VALUE` is always a string or `[[null]]` (never
 encoded/packed).

**NOTE:** Numeric measurements which collide on TIMESTAMP/UUID/NAME will store
the largest absolute value for that time period, by default. This behavior is
[configurable](../configuration.md#rawdatabase-conflictresolver).

A sample `M` record:
```
M	1512691226.137	example.com`http`c_123_987654::http`1b988fd7-d1e1-48ec-848e-55709511d43f	duration	I	1
```
This is a metric, `duration`, on account `123`, for the HTTP check
`1b988fd7-d1e1-48ec-848e-55709511d43f` with a TYPE of uint32 (`I`) and a VALUE
of `1`.

#### TSV Histogram Metrics

Histogram submission is similar to `M` records above, but instead of a
single-value payload, a base64-encoded serialization of the histogram structure
is used. This is referred to as an `H1` record. As with `M` records, the
components are tab-separated.
```
H1 TIMESTAMP UUID NAME HISTOGRAM
```

*TIMESTAMP* : Same as `M` above.

*UUID* : Same as `M` above.

*NAME* : Same as `M` above.

*HISTOGRAM* : A base64-encoded, serialized histogram. See the
`hist_serialize()` function in
[libcircllhist](https://github.com/circonus-labs/libcircllhist/blob/master/src/circllhist.c),
the reference implementation of histograms in Circonus.

A sample `H1` record:
```
H1	1512691200.000	example.com`ping_icmp`c_123_45678::ping_icmp`c50361d8-7565-4f04-8128-3cd2613dbc82	maximum	AAFQ/gAB
```
This is a histogram of values for the metric `maximum`, on an ICMP check for
account `123`.

### FlatBuffer Format

**URI:** /raw

**Method:** POST

**Format:**

A FlatBuffer metric payload is submitted as a `MetricList` as specified in the
[Reconnoiter FlatBuffer
source](https://github.com/circonus-labs/reconnoiter/blob/master/src/flatbuffers/metric_list.fbs).

When submitting FlatBuffer-encoded metrics, a client must set the HTTP
Content-Type header to `application/x-circonus-metric-list-flatbuffer`.
