# Tags

Tags in IRONdb are represented as `category:value pairs` that are separated by the colon (`:`) character.
Legal characters in an IRONdb tag category are defined by (perl RE syntax):

    perl -e '$valid = qr/[`+A-Za-z0-9!@#\$%^&"'\/\?\._-]/;

Tag values allow all of the above characters plus colon (`:`) and equals (`=`).

Any tag characters that do not fall into this set can still be ingested if they
are base64 encoded and passed in a special wrapper format.  More on this below.

Tags are ingested into IRONdb by placing the tags after the metric name with a 
tag separator character sequence: `|ST` and enclosed in square brackets `[]`. 
Commas separate each tag.

Examples:

    foo|ST[a:b]
    bar|ST[c:d]
    quux|ST[region:us-east-1,app:myapp]
    
Tags (including category and value) are limited to 256 characters for each tag.

Tags that contain characters outside of the acceptable set can be ingested by base64 encoding.
To store a metric like:

    foo|ST[~(category):<value>]
    
Where tilde `~`, parens `()`, and greater/less `<>` are outside of the character set you would encode
the category and value separately as base64 and enclose them in `b""`.  For example:

    foo|ST[b"fihjYXRlZ29yeSk=":b"PHZhbHVlPg=="]
    
It is always safe to encode *all* incoming tags in this way, the server will decide if the name
is safely representable without encoding and store the metric name decoded if it can.

> Note that this encoding also applies to tag searches if the search uses an unsupported character
> See [Searching Tags](api/search-tags.md)

