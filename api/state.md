# Getting Node State

This API call is for viewing the system state of the current node.

Data will be returned as a JSON document. The fields in this document are described below.

## Description of JSON document

`URI:`
/state

`Method:`
GET

`Output:`

*identity* :   The UUID that identifies this node.

*current* :   The current topology in which this node resides.

*next* :   The next topology for this node.

*base_rollup* :   The smallest period that is used for rolling up numeric data.

*rollups* :   An array containing a list of all data periods stored on this node.

*nnt* :   A container with information about NNT (Numeric) data storage.

* *rollups* :   An array containing a list of all NNT (numeric) data periods stored on this node.

* *rollup_&lt;period&gt;* :   Data for each particular rollup. There will be one of these entries per rollup period.

  * *fs* :   Information about file system storage for this rollup.

    * *id* :   The ID for this file system.

    * *totalMB* :   Megabytes of data used for this file system.

    * *freeMB* :   Megabytes of data available for this file system.

  * *put.calls* :   The number of put calls for this numeric period.

  * *put.elapsed_us* :   The number of microseconds spent putting data for this numeric period.

  * *get.calls* :   The number of get calls for this numeric period.

  * *get.proxy_calls* :   The number of proxy get calls for this numeric period.

  * *get.count* :   The number of metrics retrieved for this numeric period.

  * *get.elapsed_us* :   The number of microseconds spent getting data for this numeric period.

  * *extend.calls* :   The number of extend calls for this numeric period. (The number of times the system needed to extend an NNT storage file from the beginning.)

* *aggregate* :   Aggregated data from all NNT (numeric) calls. Fields are the same as for each individual rollup.

*nnt_cache_size* :   The current size of the NNT file handle cache. When IRONdb opens an NNT file for reading or writing, it will maintain an open file descriptor for it and store it in a cache.

*text* :   A container with information about text data storage.

* *fs* :   Information about file system storage for text data.

 * *id* :   The ID for this file system.

 * *totalMB* :   Megabytes of data used for this file system.

 * *freeMB* :   Megabytes of data available for this file system.

* *get* :   A container with information about text get calls.

 * *proxy_calls* :   The number of text get proxy calls.

 * *err* :   The number of text get errors.

 * *calls* :   The number of text get calls.

 * *tuples* :   The number of text get tuples.

 * *elapsed_us* :   The number of microseconds spent getting text data.

* *put* :   A container with information about text put calls.

 * *err* :   The number of text put errors.

 * *calls* :   The number of text put calls.

 * *tuples* :   The number of text put tuples.

 * *elapsed_us* :   The number of microseconds spent putting text data.

*histogram* :   A container with information about histogram data storage.   

* *rollups* :   An array containing a list of all histogram data periods stored on this node.

* *rollup_&lt;period&gt;* :   This describes data for each particular rollup. There will be one of these entries per rollup period.   

 * *fs* :   This describes information about file system storage for this rollup.

   * *id* :   The ID for this file system.

   * *totalMB* :   Megabytes of data used for this file system.

   * *freeMB* :   Megabytes of data available for this file system.

 * *put.calls* :   The number of put calls for this histogram period.

 * *put.elapsed_us* :   The number of microseconds spent putting data for this histogram period.

 * *get.calls* :   The number of get calls for this histogram period.

 * *get.proxy\_calls* :   The number of proxy get calls for this histogram period.

 * *get.count* :   The number of metrics retrieved for this histogram period.

 * *get.elapsed_us* :   The number of microseconds spent getting data for this histogram period.

* *aggregate* :   The aggregated data from all histogram calls. The fields
        displayed are the same as those listed for each
        individual rollup.

*rusage.utime*

:   Resource Usage: User CPU time used

*rusage.stime*

:   Resource Usage: System CPU time used

*rusage.maxrss*

:   Resource Usage: Maximum resident set size

*rusage.idrss*

:   Resource Usage: Integral shared memory size

*rusage.minflt*

:   Resource Usage: Page reclaims (soft page faults)

*rusage.majflt*

:   Resource Usage: Page faults (hard page faults)

*rusage.nswap*

:   Resource Usage: Swaps

*rusage.inblock*

:   Resource Usage: Block input operations

*rusage.oublock*

:   Resource Usage: Block output operations

*rusage.msgsnd*

:   Resource Usage: IPC messages sent

*rusage.msgrcv*

:   Resource Usage: IPC messages received

*rusage.nsignals*

:   Resource Usage: Signals received

*rusage.nvcsw*

:   Resource Usage: Voluntary context switches

*rusage.nivcsw*

:   Resource Usage: Involuntary context switches

*max\_peer\_lag*

:   The maximum amount by which the data on this node is behind any
    of the other Snowth nodes.

*avg\_peer\_lag*

:   The average amount by which the data on this node is behind any
    of the other Snowth nodes.

*features*

:   The features that are enabled on this node.

*text:store*

:   Appears if text data storage is enabled on this node.

*histogram:store*

:   Appears if histogram data storage is enabled on this node.

*nnt:second\_order*

:   Appears if second order derivatives for numeric data is
        enabled on this node.

*histogram:dynamic\_rollups*

:   Appears if dynamic histogram rollups are enabled on this node.

*nnt:store*

:   Appears if numeric data storage is enabled on this node.

*features*

:   Appears if feature flagging is enabled on this node.

*version*

:   The version of the Snowth software running on this node.

*application*

:   The name of this application.

## Examples

The URI "\/state" could yield the following example output:

```
{
 "identity":"5b5ea6c5-bd98-4293-91ee-ecbdf9bfb0cc",
 "current":"012345678901234567890123456789012345678901234567890123456
7890abcd",
 "next":"-",
 "base_rollup":300,
 "rollups":[300,1800,10800,86400],
 "nnt":{
  "rollups":[300,1800,10800,86400],
  "rollup_300":{
    "fs": {
      "id": 12345678,
      "totalMb": 198981.204590,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.calls":0,
   "get.proxy_calls":0,
   "get.count":0,
   "get.elapsed_us":0,
   "extend.calls":0
  },
  "rollup_1800":{
    "fs": {
      "id": 12345678,
      "totalMb": 198981.204590,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.calls":252,
   "get.proxy_calls":0,
   "get.count":88140,
   "get.elapsed_us":8342,
   "extend.calls":0
  },
  "rollup_10800":{
    "fs": {
      "id": 12345678,
      "totalMb": 198981.204590,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.calls":0,
   "get.proxy_calls":0,
   "get.count":0,
   "get.elapsed_us":0,
   "extend.calls":0
  },
  "rollup_86400":{
    "fs": {
      "id": 12345678,
      "totalMb": 198981.204590,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.calls":0,
   "get.proxy_calls":0,
   "get.count":0,
   "get.elapsed_us":0,
   "extend.calls":0
  },
  "aggregate":{
   "put.calls":0,
   "put.elapsed_us":0,
   "get.calls":255,
   "get.proxy_calls":0,
   "get.count":118653,
   "get.elapsed_us":8829,
   "extend.calls":0
  }
 },
 "nnt_cache_size":76,
 "text":{
  "fs": {
   "id": 12345678,
   "totalMb": 182607.098633,
   "availMb": 182572.629395
  },
  "get": {
    "proxy_calls":0,
     "err":0,
    "calls":9,
    "tuples":9,
    "elapsed_us":45997
  },
  "put": {
    "err":0,
    "calls":0,
    "tuples":0,
    "elapsed_us":0
  } },

  "histogram":{
   "rollups": [60,300,1800,10800,86400],
  "rollup_60":{
    "fs": {
      "id": 12345678,
      "totalMb": 186335.980957,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":0,
   "get.count":0,
   "get.elapsed_us":0
  },
  "rollup_300":{
    "fs": {
      "id": 12345678,
      "totalMb": 186335.980957,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":0,
   "get.count":0,
   "get.elapsed_us":0
  },
  "rollup_1800":{
    "fs": {
      "id": 12345678,
      "totalMb": 186335.980957,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":29,
   "get.count":8787,
   "get.elapsed_us":173635
  },
  "rollup_10800":{
    "fs": {
      "id": 12345678,
      "totalMb": 186335.980957,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":1,
   "get.count":1,
   "get.elapsed_us":13427
  },
  "rollup_86400":{
    "fs": {
      "id": 12345678,
      "totalMb": 186335.980957,
      "availMb": 182572.629395
    },
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":0,
   "get.count":0,
   "get.elapsed_us":0
  },
  "aggregate":{
   "put.calls":0,
   "put.elapsed_us":0,
   "get.proxy_calls":0,
   "get.calls":30,
   "get.count":8788,
   "get.elapsed_us":187062
  }
},
 "rusage.utime":9.716393,
 "rusage.stime":6.044374,
 "rusage.maxrss":0,
 "rusage.idrss":0,
 "rusage.minflt":0,
 "rusage.majflt":0,
 "rusage.nswap":0,
 "rusage.inblock":0,
 "rusage.oublock":0,
 "rusage.msgsnd":23754,
 "rusage.msgrcv":22521,
 "rusage.nsignals":0,
 "rusage.nvcsw":330493,
 "rusage.nivcsw":2614,
 "max_peer_lag":0,
 "features": {
 "text:store":"1",
 "histogram:store":"1",
 "nnt:second_order":"1",
 "histogram:dynamic_rollups":"1",
 "nnt:store":"1",
 "features":"1"
 },
 "version":"v307e4ccc03e73edee27f86da8d4faecd3220a93c\/e63e3b5158b8508cef6a932df33279be43d1362c",
 "application":"snowth"
}
```

