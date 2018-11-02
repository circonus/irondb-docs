# Searching tag data

Find metrics using boolean tag search.  Output is a JSON array of objects.

## Description

### URIs

* `/find/<account_id>/tags?query=<query>&activity_start_secs=<start>&activity_end_secs=<end>`
* `/find/<account_id>/tag_cats?query=<query>`
* `/find/<account_id>/tag_vals?query=<query>`

### Method

GET

### Inputs

 * `account_id`          : The account to search
 * `query`               : See [Tag Queries](/tag_queries.md) for more info on tag queries.
 * `activity_start_secs` : (optional) The start time from which to pull data, represented in seconds since the unix epoch.
 * `activity_end_secs`   : (optional) The end time up to which data is pulled, represented in seconds since the unix epoch.

## Output

### `/find/174/tags?query=and(__name:foo)`

Return all data about the incoming query.

```json
[
  {
    "uuid": "9aae16cd-4427-4330-8bd8-5c4cd176e67e",
    "check_name": "some name here",
    "metric_name": "foo|ST[app:myapp,region:us-east-1]",
    "category": "reconnoiter",
    "type": "numeric",
    "account_id": 174
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
