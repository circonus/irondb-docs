## Graphite Ingestion

There are 2 methods for ingesting graphite data into IRONdb.  

1. RESTful POST of buffers of data
2. Network socket listener akin to the normal graphite network socket listener

In both cases ASCII data in the normal graphite format are accepted:

`dot.separated.metric.name<space>12345.56<space>1480371755\n`
  
IRONdb does support a variant of the unix epoch timestamp (3rd field)
where you can suffix the timestamp with a period followed by the
number of milliseconds in the second if you desire higher resolution
data capture.

`dot.separated.metric.name<space>12345.56<space>1480371964.123\n`

The above means `123 milliseconds` into the timestamp `1480371964` or
`November 28, 2016 10:26:04 and 123ms PM UTC`

** Note that, while resembling a floating point number, this is not a float. **

For reasons of data safety we recommend that you use the RESTful POST
interface to ingest graphite data.  The network socket listener
provides no feedback to the sender about whether or not data was
actually ingested (or indeed even made it off the sender machine and
was not stuck in an outbound socket buffer) because there is no
acknowlegement mechanism on a raw socket.

The HTTP interface, on the other hand, will provide feedback about
whether data was safely ingested and will not respond until data has
actually been written by the underlying database.

Namespacing
===========

Both of the interfaces require you to namespace your graphite data.
This lets you associate a UUID/Name and numeric identifier with the
incoming metrics.  This is useful, for example, if you want to use a
single IRONdb installation to service multiple different internal
groups in your organization but keep metrics hidden across the various
groups.

All metrics live under a numeric identifier (you can think of this
like an account_id).  Metric names can only be associated with an
"account_id".  This allows you have separate graphite-web or Grafana
instances that segregate queries for metric names, or, combine them
all together under a single "account_id", or even, separate your
internal groups but recombine them under graphite-web/Grafana for
visualization purposes.  It's really up to you.

Further, IRONdb requires associating incoming graphite data with a
UUID and Name to make Graphite data match reconnoiter ingested data
more closely on the Circonus platform.  We hide the complexity of this
on the rendering side so you only have to worry about this mapping on
the ingestion side.

Writing Graphite Data with HTTP
===============================

Graphite data is sent as buffers of N rows of graphite formatted data
to the graphite ingestion endpoint:

`http://<snowth_machine:port>/graphite/<account_id>/<uuid>/<check_name>`

For example:

`http://<snowth_machine:port>/graphite/1/8c01e252-e0ed-40bd-d4a3-dc9c7ed3a9b2/dev`

Will place all metrics under account_id `1` with that UUID and call them `dev`.

`http://<snowth_machine:port>/graphite/1/45e77556-7a1b-46ef-f90a-cfa34e911bc3/prod`

Will place all metrics under account_id `1` with that UUID and call them `prod`.

This is important later when we render the metrics in the UI (see
reading-graphite.md for more info).  Metrics ingested under example 1
will render as:

`graphite.dev.metric.name.here`

And the second example:

`graphite.prod.metric.name.here`


Writing Graphite Data with Network Listener
===========================================
 
The network listener requires that we associate an account_id, uuid
and name with a network port.  We do this via the IRONdb configuration
file by adding a new listener stanza:

```
  <listeners>
    <listener address="*" port="2003" type="graphite">
      <config>
        <check_uuid>8c01e252-e0ed-40bd-d4a3-dc9c7ed3a9b2</check_uuid>
        <check_name>dev</check_name>
        <account_id>1</account_id>
      </config>
    </listener>
  </listeners>
```

This listener stanza is the same as the first example under the HTTP
ingestion section.  You can then:

```
echo "my.metric.name.one 1 `date +%s`" | nc 2003
```
To send metrics to IRONdb.

