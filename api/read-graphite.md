# Retrieving Graphite Data

Fetches Graphite-style data.  This is similar to the [rollup](read-rollup.md)
endpoint but the data returned is always **average** data and this endpoint
will scale the `rollup_span` to match the time range of data requested.

See [graphite rendering](/graphite-rendering.md#get).

