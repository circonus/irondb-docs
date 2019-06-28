# Graphite Integration

IRONdb is a drop-in replacement for Graphite's Whisper database, and supports
ingestion from Carbon sources like carbon-relay and carbon-c-relay.
[Graphite-irondb](https://github.com/circonus-labs/graphite-irondb) is a
storage finder plugin that allows IRONdb to seamlessly integrate with an
organization's existing Graphite-web deployment.

The [IRONdb Relay](irondb-relay.md) is a scalable, drop-in replacement for
carbon-relay or carbon-c-relay.

The following pages detail how to write, search, and retrieve Graphite data
with IRONdb.
* [Writing Graphite Metrics](graphite-ingestion.md)
* [Searching and Rendering Graphite Metrics](graphite-rendering.md)
