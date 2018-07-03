# Activity Tracking

Starting with release [0.12](/changelog.md), IRONdb supports tracking of metric activity without the expense
of reading all known time series data to find active ranges.  The activity of a metric is tracked at a 5
minute granularity.  Any ingestion of a metric will mark that 5 minute period that the timestamp falls
into as active for that metric.  

This activity tracking also coalesces nearby active ranges.  Any activity on a metric within an 8 hour
window marks that metric as active for that 8 hour span.  For example, if you have a metric that arrived
with the timestamp: `2018-07-03T11:00:01:123Z` and then nothing else arrived until `2018-07-03T19:00:02:123Z`, 
the metric would be considered *inactive* in the 8 hour span between these 2 timestamps.   If, later, some
late data arrives and we see a timestamp at: `2018-07-03T14:00:01:123Z`, then the entire 8 hour span is 
considered *active* for purposes of querying.

See [Searching Tags](api/search-tags.md#inputs) on how to query activity
periods for a given list of metrics.

This activity tracking only applies to data ingested after the upgrade to `0.12` or later.  Any data
ingested prior to installation of `0.12` will be invisible to the activity tracking code.  However,
IRONdb also ships with an API to rebuild activity tracking data by reading the actual datapoints for a
metric to determine its activity ranges.  Since this is an expensive operation it has to be triggered 
for a list of metrics by an operator.  

## Rebuilding Activity Data 

> Do not trigger this API until you have upgraded all IRONdb nodes to `0.12` or later.

### URI

* `/surrogate/activity_rebuild`

### Method

POST

### Inputs

A JSON document which lists the set of metrics to rebuild activity data for, with the syntax:

```json
[
   {
      "check_uuid":"1fd7c873-0055-4bd3-a16a-2137b111e71a",
      "metric_name":"foo"
   },
   {
      "check_uuid":"1fd7c873-0055-4bd3-a16a-2137b111e71a",
      "metric_name":"bar"
   },
   {
      "check_uuid":"1fd7c873-0055-4bd3-a16a-2137b111e71a",
      "metric_name":"baz|ST[a:b,c:d]"
   }
]
```
    
> The above will rebuild activity for the 3 metrics listed in the document.

