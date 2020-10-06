.. _PMWEBAPI:


PMWEBAPI
###########

.. contents::


+-------------------+--------------------------------------------------------------------------------------------------------------------------+
| **NAME**          | **PMWEBAPI**- Introduction to the Performance Metrics Web Application Programming Interface                              |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
| **HTTP SYNOPSIS** | **GET /metrics**                                                                                                         |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
|                   | **GET /series/...**                                                                                                      |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
|                   | **GET /search/...**                                                                                                      |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
|                   | **GET /pmapi/...**                                                                                                       |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
| **C SYNOPSIS**    | #include< `pcp/pmwebapi.h <https://github.com/performancecopilot/pcp/blob/master/src/include/pcp/pmwebapi.h>`_ >         |  
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
|                   | ... assorted routines ...                                                                                                |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+
|                   | cc ... -lpcp_web -lpcp                                                                                                   |
+-------------------+--------------------------------------------------------------------------------------------------------------------------+

**DESCRIPTION**

The PMWEBAPI is a collection of interfaces providing Performance Co-Pilot services for web applications. It consists of APIs for web applications querying and 
analysing both live and historical performance data, as well as APIs used by web servers.

The usual HTTP URL-encoded optional parameter rules apply and PMWEBAPI REST requests always follow the convention:

*/api/endpoint ? parameter1 = value1 & parameter2 = value2*

Examples in all following sections use the curl (1) command line utility with a local `pmproxy <https://pcp.io/man/man1/pmproxy.1.html>`_ (1) server listening on 
port 44322 (default port). The `pmjson <https://pcp.io/man/man1/pmjson.1.html>`_ (1) utility is used to neatly format any JSON output, as opposed to the compact 
(minimal whitespace) form provided by default. The examples in the scalable time series section use historical data recorded by the 
`pmlogger <https://pcp.io/man/man1/pmlogger.1.html>`_ (1) service, in conjunction with a local redis-server (1).

OPEN METRICS
**************

Exporting of live performance metrics in an Open Metrics compatible format (as described at https://openmetrics.io and popularized by the 
https://prometheus.io project) is available.

GET /metrics
============

+---------------+--------------+--------------------------------------------------------------------------------------------+
| Parameters    | Type         | Explanation                                                                                |
+===============+==============+============================================================================================+
| names	        | string       | Comma-separated list of metric names                                                       |
+---------------+--------------+--------------------------------------------------------------------------------------------+
| times	        | boolean      | Append sample times (milliseconds since epoch)                                             |
+---------------+--------------+--------------------------------------------------------------------------------------------+

Fetches current values and metadata for all metrics, or only metrics indicated by a comma-separated list of *names*.

For all numeric metrics with the given NAME prefixes, create an Open Metrics (Prometheus) text export format giving their current value and related metadata.

The response has plain text type rather than JSON commonly used elsewhere in the REST API. This format can be injested by many open source monitoring tools, 
including Prometheus and `pmdaopenmetrics <https://pcp.io/man/man1/pmdaopenmetrics.1.html>`_(1).

The native PCP metric metadata (metric name, type, indom, semantics and units) is first output for each metric with **# PCP** prefix. The metadata reported is of 
the form described on `pmTypeStr <https://pcp.io/man/man3/pmtypestr.3.html>`_ (3), `pmInDomStr <https://pcp.io/man/man3/pmindomstr.3.html>`_ (3), 
`pmSemStr <https://pcp.io/man/man3/pmsemstr.3.html>`_ (3) and `pmUnitsStr <https://pcp.io/man/man3/pmunitsstr.3.html>`_ (3) respectively. If the 
`pmUnitsStr <https://pcp.io/man/man3/pmunitsstr.3.html>`_ (3) units string is empty, then **none** is output. The units metadata string may contain spaces and 
extends to the end of the line.

PCP metric names are mapped so that the **.** separators are exchanged with **_** (':' in back-compatibility mode, where "# PCP" is the identifying line suffix). 
Both metric labels and instances are represented as Prometheus labels, with external instance names being quoted and the flattened PCP metric hierarchy being 
presented with each value.

.. sourcecode:: none

 $ curl -s http://localhost:44322/metrics?\
         names=proc.nprocs,kernel.pernode.cpu.intr,filesys.blocksize
 
 # PCP5 proc.nprocs 3.8.99 u32 PM_INDOM_NULL instant none
 # HELP proc_nprocs instantaneous number of processes
 # TYPE proc_nprocs gauge
 proc_nprocs {hostname="app1"} 7
 
 # PCP5 kernel.pernode.cpu.intr 60.0.66 u64 60.19 counter millisec
 # HELP kernel_pernode_cpu_intr total interrupt CPU [...]
 # TYPE kernel_pernode_cpu_intr counter
 kernel_pernode_cpu_intr{hostname="app1",instname="node0"} 25603
 
 # PCP5 filesys.blocksize 60.5.9 u32 60.5 instant byte
 # HELP filesys_blocksize Size of each block on mounted file[...]
 # TYPE filesys_blocksize gauge
 filesys_blocksize{hostname="app1",instname="/dev/sda1"} 4096
 filesys_blocksize{hostname="app1",instname="/dev/sda2"} 4096

SCALABLE TIME SERIES
**********************

The fast, scalable time series query capabilities provided by the `pmseries <https://pcp.io/man/man1/pmseries.1.html>`_ (1) command are also available through a 
REST API. These queries provide access to performance data (metric metadata and values) from multiple hosts simultaneously, and in a fashion suited to efficient 
retrieval by any number of web applications.

All requests in this group can be accompanied by an optional *client* parameter. The value passed in the request will be sent back in the response - all responses 
will be in JSON object form in this case, with top level "client" and "result" fields.

GET */series/query* - `pmSeriesQuery <https://pcp.io/man/man3/pmseriesquery.3.html>`_ (3)
=============================================================================================

+---------------+--------------+--------------------------------------------------------------------------------------------+
| Parameters    | Type         | Explanation                                                                                |
+===============+==============+============================================================================================+
| expr	        | string       | Query string in `pmseries <https://pcp.io/man/man1/pmseries.1.html>`_ (1) format           |        
+---------------+--------------+--------------------------------------------------------------------------------------------+
| client        | string       | Request identifier sent back with response                                                 |
+---------------+--------------+--------------------------------------------------------------------------------------------+

Performs a time series query for either matching identifiers, or matching identifiers with series of time-stamped values.

The query is in the format described in `pmseries <https://pcp.io/man/man1/pmseries.1.html>`_ (1) and is passed to the server via either the *expr* parameter 
(HTTP GET) or via the message body (HTTP POST).

When querying for time series matches only, no time window options are specified and matching series identifiers are returned in a JSON array.

.. code-block:: none

 $ curl -s http://localhost:44322/series/query?\
         expr=disk.dev.read* | pmjson

.. code-block:: JSON

 [
   "9d8c7fb51ce160eb82e3669aac74ba675dfa8900",
   "ddff1bfe286a3b18cebcbadc1678a68a964fbe9d",
   "605fc77742cd0317597291329561ac4e50c0dd12"
 ]

When querying for time series values as well, a time window must be specified as part of the query string. The simplest form is to just request the most recent 
sample.

.. code-block:: none

 $ curl -s http://localhost:44322/series/query?\
         expr=disk.dev.read*[samples:1] | pmjson

.. code-block:: JSON

 [
   {
     "series": "9d8c7fb51ce160eb82e3669aac74ba675dfa8900",
     "instance": "c3795d8b757506a2901c6b08b489ba56cae7f0d4",
     "timestamp": 1547483646.2147431,
     "value": "12499"
   }, {
     "series": "ddff1bfe286a3b18cebcbadc1678a68a964fbe9d",
     "instance": "6b08b489ba56cae7f0d4c3795d8b757506a2901c",
     "timestamp": 1547485701.7431218,
     "value": "1118623"
   }, {
     "series": "605fc77742cd0317597291329561ac4e50c0dd12",
     "instance": "c3795d8b757506a2901c6b08b489ba56cae7f0d4",
     "timestamp": 1547483646.2147431,
     "value": "71661"
   }
 ]

GET */series/values* - pmSeriesValues (3)
==========================================

+---------------+--------------+----------------------------------------------------+
| Parameters    | Type         | Explanation                                        |
+===============+==============+====================================================+
| series    	| string       | Comma-separated list of series identifiers         |
+---------------+--------------+----------------------------------------------------+
| client        | string       | Request identifier sent back with response         |
+---------------+--------------+----------------------------------------------------+
| samples       | number       | Count of samples to return                         |
+---------------+--------------+----------------------------------------------------+
| interval      | string       | Time between successive samples                    |
+---------------+--------------+----------------------------------------------------+
| start	        | string       | Sample window start time                           |
+---------------+--------------+----------------------------------------------------+
| finish        | string       | Sample window end time                             |
+---------------+--------------+----------------------------------------------------+
| offset        | string       | Sample window offset                               |
+---------------+--------------+----------------------------------------------------+
| align	        | string       | Sample time alignment                              |
+---------------+--------------+----------------------------------------------------+
| zone	        | string       | Time window timezone                               |
+---------------+--------------+----------------------------------------------------+

Performs values retrievals for one or more time series identifiers. The JSON response contains the same information as the **pmseries - q /-- query** option 
using any of the time window parameters described on `pmseries <https://pcp.io/man/man1/pmseries.1.html>`_ (1). If no time window parameters are specified, the 
single most recent value observed is retrieved.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/values?\
         series=605fc77742cd0317597291329561ac4e50c0dd12 | pmjson

.. sourcecode:: JSON

 [
   {
     "series": "605fc77742cd0317597291329561ac4e50c0dd12",
     "timestamp": 1317633022959.959241041,
     "value": "71660"
   }
 ]

GET */series/descs* - `pmSeriesDescs <https://pcp.io/man/man3/pmseriesdescs.3.html>`_ (3)
============================================================================================

+---------------+--------------+----------------------------------------------------+
| Parameters    | Type         | Explanation                                        |
+===============+==============+====================================================+
| series        | string       | Comma-separated list of series identifiers         |
+---------------+--------------+----------------------------------------------------+
| client        | string       | Request identifier sent back with response         |
+---------------+--------------+----------------------------------------------------+

Performs a descriptor lookup for one or more time series identifiers. The JSON response contains the same information as the **pmseries - d /-- desc** option.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/descs?\
         series=605fc77742cd0317597291329561ac4e50c0dd12 | pmjson

.. sourcecode:: JSON

 [
   {
     "series": "605fc77742cd0317597291329561ac4e50c0dd12",
     "source": "f5ca7481da8c038325d15612bb1c6473ce1ef16f",
     "pmid": "60.0.4",
     "indom": "60.1",
     "semantics": "counter",
     "type": "u32",
     "units": "count",
   }
 ]

GET */series/labels* - pmSeriesLabels (3), pmSeriesLabelValues (3)
=====================================================================

+---------------+--------------+----------------------------------------------------+
| Parameters    | Type         | Explanation                                        |
+===============+==============+====================================================+
| series        | string       | Comma-separated list of series identifiers         |
+---------------+--------------+----------------------------------------------------+
| match	        | string       | Glob pattern string to match on all labels         |
+---------------+--------------+----------------------------------------------------+
| name	        | string       | Find all known label values for given name         |
+---------------+--------------+----------------------------------------------------+
| names	        | string       | Comma-separated list of label names                |
+---------------+--------------+----------------------------------------------------+
| client        | string       | Request identifier sent back with response         |
+---------------+--------------+----------------------------------------------------+

This command operates in one of three modes. It can perform a label set lookup for one or more time series identifiers, when given the *series* parameter). It can 
produce a list of all known label names, in the absense of *name* , *names* or *series* parameters. The JSON responses for these modes contains the same information 
as the **pmseries - l /-- labels** option.

Alternatively, it can produce a list of all known label values for a given label *name* or *names* . The JSON response for this mode contains the same information 
as the **pmseries - v /-- values** option.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/labels?\
         series=605fc77742cd0317597291329561ac4e50c0dd12 | pmjson

.. sourcecode:: JSON

 [
   {
     "series": "605fc77742cd0317597291329561ac4e50c0dd12",
     "labels": {
       "agent": "linux",
       "domainname": "acme.com",
       "groupid": 1000,
       "hostname": "`www.acme.com <http://www.acme.com/>`_",
       "latitude": -25.28496,
       "longitude": 152.87886,
       "machineid": "295b16e3b6074cc8bdbda8bf96f6930a",
       "platform": "dev",
       "userid": 1000
     }
   }
 ]

Alternatively, with no *name* , *names* or *series* parameters, return the list of all known label names.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/labels | pmjson

.. sourcecode:: JSON

 [
     "agent",
     "appversion",
     "domainname",
     "groupid",
     "hostname",
     "jobid",
     "latitude",
     "longitude",
     "machineid",
     "platform",
     "userid"
 ]

Use the *name* or *names* parameters to find all possible label values for the given name(s).

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/labels?\
         names=hostname,domainname | pmjson

.. sourcecode:: JSON

 {
     "hostname": [ "app", "nas" ],
     "domainname": [ "acme.com" ]
 }
 
GET */series/metrics* - pmSeriesMetrics (3)
===============================================

+---------------+--------------+----------------------------------------------------+
| Parameters    | Type         | Explanation                                        |
+===============+==============+====================================================+
| series        | string       | Comma-separated list of series identifiers         |
+---------------+--------------+----------------------------------------------------+
| match	        | string       | Glob pattern string to match on all names          |
+---------------+--------------+----------------------------------------------------+
| client        | string       | Request identifier sent back with response         |
+---------------+--------------+----------------------------------------------------+

Performs a metric name lookup for one or more time series identifiers. The JSON response contains the same information as the **pmseries - m /-- metrics** option.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/metrics?\
         series=605fc77742cd0317597291329561ac4e50c0dd12 | pmjson

.. sourcecode:: JSON

 [
   {
     "series": "605fc77742cd0317597291329561ac4e50c0dd12",
     "name": "disk.dev.read_bytes"
   }
 ]

Alternatively, with no *series* argument, this request will return the list of all known metric names.

.. sourcecode:: none

 $ curl -s http://localhost:44322/series/metrics | pmjson

.. sourcecode:: JSON

 [
     "disk.dev.read",
     "disk.dev.read_bytes",
     "disk.dev.read_merge",
     "kernel.all.load",
     "kernel.all.pswitch",
 ]

