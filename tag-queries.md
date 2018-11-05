# Tag Queries

Tag queries can be used to find or perform deletion of metrics using a boolean tag search.

See [Tag Support](/tags.md) for more info on tag formats.


## Query syntax

A query follows this eBNF syntax:

    query-param = all-of | any-of | not
    all-of = "and(" query-tag-list ")"
    any-of = "or(" query-tag-list ")"
	not = "not(" query-tag-el ")"
    query-tag-list = query-tag-el | query-tag-el "," query-tag-list
    query-tag-el = all-of | any-of | not | tag-category:tag-value | /cat regex/:/val regex/ | glob

A `not` clause may only contain a single expression, whereas `and`/`or` may each contain a list of expressions.
Each expression may be a literal `key:value` to match, a regular expression, or a glob match syntax.

Regular expressions follow the PCRE2 syntax and are of the form:

    /category regex/:/value regex/

Note that you can apply regular expressions independently to category or value or both:

    category:/value regex/
    /category regex/:value

Glob syntax supports the wildcard "`*`" and can be used as a completer:

    categ*:value
    category:val*
    *:*

The last will match every tag and pull everything for the account.

There are 2 special tags:

* `__name`
* `__check_uuid`

Which do not explicitly appear in metric names but can be used to find metrics
anyway. For example, you could query activity periods for all metrics within a
given `__check_uuid` even if none of those metrics were submitted with tags.

If your query uses an unsupported tag character (see [Tag Support](/tags.md)) you must
enclose the query in base64 notation:

`and(b"...":b"...")`

To pass through the unsupported characters. If using regular expression
patterns, the `/ /` do _not_ need to be encoded. To perform a regex match on
`.*foo`, you would use the form `b:/Lipmb28K/`.

See the examples below for more color.


## Query Examples

You have ingested the following metrics:

    foo|ST[region:us-east-1,app:myapp]
    bar|ST[region:us-east-2,app:myapp]
    baz|ST[region:us-west-1,app:myapp]
    quux|ST[region:us-west-2,app:yourapp]

To find all of the metrics under `app:myapp` your query would be:

`and(app:myapp)`

To find all of the metrics in `us-east` regardless of sub-region you would do:

`and(region:us-east-*)` in glob syntax or:

`and(region:/us-east-.*/)` in regex syntax.

To find `bar` or `quux` you could either do:

`or(__name:bar,__name:quux)`

or:

`or(and(region:us-east-2,app:,myapp),and(region:us-west-2,app:yourapp))`
