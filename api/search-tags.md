# Searching tag data

Find metrics using boolean tag search.  Output is a JSON array of objects.

## Description

### URIs

* `/find/<account_id>/tags?query=<query>&activity_start_secs=<start>&activity_end_secs=<end>&latest=<0|1|2>`
* `/find/<account_id>/tag_cats?query=<query>`
* `/find/<account_id>/tag_vals?query=<query>`

### Method

GET

### Inputs

 * `account_id`          : The account to search
 * `query`               : See [Tag Queries](/tag-queries.md) for more info on tag queries.
 * `activity_start_secs` : (optional) The start time from which to pull data, represented in seconds since the unix epoch.
 * `activity_end_secs`   : (optional) The end time up to which data is pulled, represented in seconds since the unix epoch.
 * `latest`              : (optional, default 0) Specify if the latest values for the metric should be returned.  Parameters:
   *  0 : Do not return latest values.
   *  1 : Return latest values if it is a no-work operation
   *  2 : Return latest values even if work must be performed, and turn on tracking for this metric so it will be "free" for later calls.

## Output

### `/find/174/tags?query=and(__name:foo)`

Return all metrics matching a tag query along with information about those metrics.  If [activity tracking](../activity_tracking.html) is turned on this will include activity windows for the metric.  If latest value tracking is turned on and requested, this will include the 2 most recent value tuples for the metric, if available.

```json
[
  {
    "uuid": "9aae16cd-4427-4330-8bd8-5c4cd176e67e",
    "check_name": "some name here",
    "metric_name": "foo|ST[app:myapp,region:us-east-1]",
    "category": "reconnoiter",
    "type": "numeric",
    "account_id": 174,
    "activity": [
      [ 1558029600, 1558032300 ],
      [ 1559746800, 1569273300 ]
    ],
    "latest": {
      "numeric": [
        [ 1569271882337, 2991012437 ],
        [ 1569271942930, 2991020000 ]
      ]
    }
  }
]

```

### `/find/174/tag_cats?query=and(__name:foo)`

Return the categories of the incoming query.

```json
[
   "app",
   "region"
]

```

### `/find/174/tag_vals?query=and(__name:foo)`

Return the values of the incoming query.

```json
[
   "myapp",
   "us-east-1"
]

```
