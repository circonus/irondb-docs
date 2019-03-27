# Wildcard, Tag Query and Check Delete Result Statuses

When doing a delete which could affect multiple metrics, the returned JSON response will indicate the final status for each metric which matched the request.  A list of these statuses and a description is given below.  Note that, in many cases, the "payload' field will contain further details.

 * `Bad request` :     The URI did not conform to expected syntax or inputs for the API
 * `Deleted` :         Data was found and the deletion completed successfully
 * `Found` :           Data was found that can be deleted if request is submitted again with delete confirmation
 * `Invalid range` :   An argument is not within the proper range of allowable values
 * `No content` :      No data to be deleted was found (prior to the end time if not full delete)
 * `Not found` :       The metric name was not found
 * `Not implemented` : The supplied request is not currently implemented
 * `Not local` :       The metric's data is not stored or replicated on this node of the cluster
 * `Redirected` :      The request for deletion was forwarded to another node(s)
 * `Server error` :    An error occurred while performing the deletion
 * `Unable busy` :     The deletion request cannot be performed currently, please try later
 * `Undefined` :       The result code is unknown and not valid

For operations that utilized a pattern match or tag query, and also set the
`x-snowth-advisory-limit` header, if the result set reached the limit, a
response header, `x-snowth-results-limited`, will be returned with a value of
1.
