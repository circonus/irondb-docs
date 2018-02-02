Deleting All Data Before A Date
==================================

This API call is for deleting any type of data from the IRONdb cluster prior to
a given date, irrespective of check or metric. This is typically used for
cluster-wide retention policies.

This call will return an empty array upon success. If there is an error, this
call will return a JSON object with the error.

Description of API call
-----------------------

**URI:**   /sweep\_delete

**Method:**   DELETE

**Inputs:**   

None. All options are provided by Headers (see below).

**Headers:**

The timestamp is provided via a header:

X-Snowth-Delete-Time: &lt;new\_epoch&gt;

The type of data (numeric, text, histogram) to remove may be specified, with
multiple values comma-separated. If this header is not specified, only numeric
data (nnt) will be removed.

X-Snowth-Delete-Data-Types: nnt,text,histogram

Specific NNT rollup spans may be removed (all spans will be removed if not
specified), with multiple values comma-separated. The rollup span(s) must match
the period values configured in `irondb.conf`, in the `<rollups>` stanza.

X-Snowth-NNT-Delete-Rollups: &lt;rollup\_span\_in\_seconds&gt;

Specific histogram rollup spans may be removed (all spans will be removed if not
specified), with multiple values comma-separated. The rollup span(s) must match
the period values configured in `irondb.conf`, in the `<histogram>` stanza.

X-Snowth-Histogram-Delete-Rollups: &lt;rollup\_span\_in\_seconds&gt;

Text metrics do not have the concept of rollups, so specifying the delete time
and data type are sufficient to remove all text data prior to the given time.

Examples
--------

Delete all 1-minute numeric rollups older than 2017-06-01

```
curl -X DELETE \
  -H 'X-Snowth-Delete-Time: 1496275200' \
  -H 'X-Snowth-Delete-Data-Types: nnt' \
  -H 'X-Snowth-NNT-Delete-Rollups: 60' \
  http://127.0.0.1:8112/sweep_delete
```

Delete data of all types and all rollup spans older than 2017-06-01

```
curl -X DELETE \
  -H 'X-Snowth-Delete-Time: 1496275200' \
  -H 'X-Snowth-Delete-Data-Types: nnt,text,histogram' \
  http://127.0.0.1:8112/sweep_delete
```
