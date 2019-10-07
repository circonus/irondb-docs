# OpenTSDB Ingestion

IRONdb has native endpoints for accepting OpenTSDB-style data.

## Enable the OpenTSDB Module

> As of version 0.17.0, the OpenTSDB module is active by default for new
> installations. If you previously activated the module using the instructions
> below, you may remove the line from `irondb-modules-site.conf` after
> upgrading to 0.17.0 or later, but it is not an error if the line appears more
> than once.

IRONdb must be [configured](configuration.html) such that the OpenTSDB module is
enabled for reading or writing OpenTSDB data natively. OpenTSDB support is
activated by adding the following line:
```
<generic image="opentsdb" name="opentsdb"/>
```
to `/opt/circonus/etc/irondb-modules-site.conf` on each IRONdb node. This file
preserves local modifications across package updates. A [service
restart](operations.md#service-management) is required after changing
configuration.

## Ingestion Format

There are 2 methods for ingesting OpenTSDB data into IRONdb:

1. RESTful HTTP POST of OpenTSDB JSON formatted datapoint(s)
2. Network socket listener akin to the normal OpenTSDB telnet method

For the HTTP method, POST a JSON object (or an array of JSON objects) to
the RESTful API endpoint (see the section below - *Writing OpenTSDB with HTTP*).
Each datapoint should be encoded as follows:

```
{
  "metric": "metric_name",
  "timestamp": timestamp,
  "value": value,
  "tags": {
    "tag_key": "tag_value",
    "tag_key2": "tag_value2",
    ...
    "tag_keyn": "tag_valuen"
  }
}
```

At least one tag key/value pair is required. Multiple datapoints can be sent
as a JSON array, separated by commas, and the entire POST enclosed in square
brackets.

For example:

```
[{
  "metric": "my.metric.name",
  "timestamp": 1544678300,
  "value": 637,
  "tags": {
    "datacenter": "east"
  }
},
{
  "metric": "myother.metric.name",
  "timestamp": 1544688100,
  "value": 3475,
  "tags": {
    "datacenter": "west"
  }
}]
```

In the case of the telnet method, telnet `put` commands in the normal OpenTSDB
format are accepted:

`put<space>metric_name<space>timestamp<space>value<space>tag_key=tag_value{<space>tag_key2=tag_value2...<space>tag_keyn=tag_valuen}`

At least one tag key/value pair must be included.  For example:

`put my.metric.name<space>1480371755<space>12345.56<space>datacenter=east`

If you desire higher resolution data capture, you can suffix the timestamp with a
period, followed by the number of milliseconds in the second, or simply just
use 13 numeric digits without the period (the last three digits will become the
millseconds). For example:

`put my.metric.name<space>1480371964.123<space>12345.56<space>datacenter=east`

Or just:

`put my.metric.name<space>1480371964123<space>12345.56<space>datacenter=east`

These two examples mean `123 milliseconds` into the timestamp `1480371964` or
`November 28, 2016 10:26:04 and 123ms PM UTC`

** Note that, while it resembles a floating point number, this is not a float. **

For data safety reasons, we recommend that you use the RESTful POST interface to
send OpenTSDB-formatted JSON data. The network socket listener provides no
feedback to the sender about whether or not data was actually ingested (or indeed
even made it off the sender machine and was not stuck in an outbound socket
buffer) because there is no acknowledgement mechanism on a raw socket.

The HTTP interface, on the other hand, will provide feedback about whether data
was safely ingested and will not respond until data has actually been written by
the underlying database.

## Namespacing

Both of the interfaces require you to namespace your OpenTSDB data. This lets
you associate a UUID/Name and numeric identifier with the incoming metrics. This
is useful, for example, if you want to use a single IRONdb installation to
service multiple different internal groups in your organization but keep metrics
hidden across the various groups.

All metrics live under a numeric identifier (you can think of this like an
account_id). Metric names can only be associated with an "account_id". This
allows you have separate client instances that segregate queries for metric
names, or combine them all together under a single "account_id", or even
separate your internal groups but recombine them under the client for
visualization purposes. It's really up to you.

Furthermore, IRONdb requires associating incoming OpenTSDB data with a UUID and
Name to make OpenTSDB data match data ingested from native sources more closely
on the Circonus platform. We hide the complexity of this on the rendering side,
so you only have to worry about this mapping on the ingestion side. This UUID
can be created using `uuidgen` on a typical UNIX(like) system or via any
external tool or website that generates [well-formed,
non-nil](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random))
UUIDs.

When we store these metric names inside IRONdb, we prefix them with our standard
collection category ("reconnoiter" will be automatically assigned) and the
"Name" of the check. You can see this in the examples below in more detail.

Adding these additional fields allow us to disambiguate metric names from
potential duplicate names collected from other sources.

## Writing OpenTSDB Data with HTTP

OpenTSDB data is sent by POSTing a JSON object or an array of JSON objects
using the format described above to the OpenTSDB ingestion endpoint:

`http://<irondb_machine:port>/opentsdb/<account_id>/<uuid>/<check_name>`

For example:

`http://192.168.1.100:4242/opentsdb/1/8c01e252-e0ed-40bd-d4a3-dc9c7ed3a9b2/dev`

This will place all metrics under account_id `1` with that UUID and call them `dev`.

`http://192.168.1.100:4242/opentsdb/1/45e77556-7a1b-46ef-f90a-cfa34e911bc3/prod`

This will place all metrics under account_id `1` with that UUID and call them `prod`.

## Writing OpenTSDB Data with Network Listener

The network listener requires that we associate an account_id, uuid, and name
with a network port. This is added to the [IRONdb configuration
file](/configuration.md) during initial installation, for the default OpenTSDB
text protocol port (4242). Additional stanzas may be added, associating
different IDs with different ports to segregate incoming traffic.

```
    <listener address="*" port="4243" type="opentsdb">
      <config>
        <check_uuid>549a90ee-c5bb-4b0f-bcb4-e942b0503f85</check_uuid>
        <check_name>myothercheckname</check_name>
        <account_id>1</account_id>
      </config>
    </listener>
```

You can then use:

```
echo "my.metric.name.one `date +%s` 1 cpu=1" | nc 4243
```

to send metrics to IRONdb, and it will store the datapoint under the supplied
metric name with the account, uuid, and name that was provided by the
configuration for the port that was used.
