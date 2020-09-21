.. _PerformanceMetricsInferenceEngine:

Performance Metrics Inference Engine
#####################################

The Performance Metrics Inference Engine (**pmie**) is a tool that provides automated monitoring of, and reasoning about, system performance within the 
Performance Co-Pilot (PCP) framework.

The major sections in this chapter are as follows:

Section 5.1, “`Introduction to pmie`_”, provides an introduction to the concepts and design of **pmie**.

Section 5.2, “`Basic pmie Usage`_”, describes the basic syntax and usage of **pmie**.

Section 5.3, “`Specification Language for pmie`_”, discusses the complete **pmie** rule specification language.

Section 5.4, “`pmie Examples`_”, provides an example, covering several common performance scenarios.

Section 5.5, “`Developing and Debugging pmie Rules`_”, presents some tips and techniques for **pmie** rule development.

Section 5.6, “`Caveats and Notes on pmie`_”, presents some important information on using **pmie**.

Section 5.7, “`Creating pmie Rules with pmieconf`_”, describes how to use the **pmieconf** command to generate **pmie** rules.

Section 5.8, “`Management of pmie Processes`_”, provides support for running **pmie** as a daemon.

.. contents::

Introduction to pmie
*********************

Automated reasoning within Performance Co-Pilot (PCP) is provided by the Performance Metrics Inference Engine, (**pmie**), which is an applied artificial 
intelligence application.

The **pmie** tool accepts expressions describing adverse performance scenarios, and periodically evaluates these against streams of performance metric 
values from one or more sources. When an expression is found to be true, **pmie** is able to execute arbitrary actions to alert or notify the system 
administrator of the occurrence of an adverse performance scenario. These facilities are very general, and are designed to accommodate the automated 
execution of a mixture of generic and site-specific performance monitoring and control functions.

The stream of performance metrics to be evaluated may be from one or more hosts, or from one or more PCP archive logs. In the latter case, **pmie** may be 
used to retrospectively identify adverse performance conditions.

Using **pmie**, you can filter, interpret, and reason about the large volume of performance data made available from PCP collector systems or PCP archives.

Typical **pmie** uses include the following:

* Automated real-time monitoring of a host, a set of hosts, or client-server pairs of hosts to raise operational alarms when poor performance is detected in a production environment

* Nightly processing of archive logs to detect and report performance regressions, or quantify quality of service for service level agreements or management reports, or produce advance warning of pending performance problems

* Strategic performance management, for example, detection of slightly abnormal to chronic system behavior, trend analysis, and capacity planning

The **pmie** expressions are described in a language with expressive power and operational flexibility. It includes the following operators and functions:

* Generalized predicate-action pairs, where a predicate is a logical expression over the available performance metrics, and the action is arbitrary. Predefined actions include the following:

  *  Launch a visible alarm with **pmconfirm**; see the **pmconfirm(1)** man page.
  *  Post an entry to the system log file; see the **syslog(3)** man page.
  *  Post an entry to the PCP noticeboard file ``${PCP_LOG_DIR}/NOTICES``; see the **pmpost(1)** man page.
  *  Execute a shell command or script, for example, to send e-mail, initiate a pager call, warn the help desk, and so on.
  *  Echo a message on standard output; useful for scripts that generate reports from retrospective processing of PCP archive logs.

* Arithmetic and logical expressions in a C-like syntax.

* Expression groups may have an independent evaluation frequency, to support both short-term and long-term monitoring.

* Canonical scale and rate conversion of performance metric values to provide sensible expression evaluation.

* Aggregation functions of **sum, avg, min**, and **max**, that may be applied to collections of performance metrics values clustered over multiple hosts, or multiple instances, or multiple consecutive samples in time.

* Universal and existential quantification, to handle expressions of the form “for every....” and “at least one...”.

* Percentile aggregation to handle statistical outliers, such as “for at least 80% of the last 20 samples, ...”.

* Macro processing to expedite repeated use of common subexpressions or specification components.

* Transparent operation against either live-feeds of performance metric values from PMCD on one or more hosts, or against PCP archive logs of previously accumulated performance metric values.

The power of **pmie** may be harnessed to automate the most common of the deterministic system management functions that are responses to changes in system performance. For example, disable a batch stream if 
the DBMS transaction commit response time at the ninetieth percentile goes over two seconds, or stop accepting uploads and send e-mail to the *sysadmin* alias if free space in a storage system falls below five 
percent.

Moreover, the power of **pmie** can be directed towards the exceptional and sporadic performance problems. For example, if a network packet storm is expected, enable IP header tracing for ten seconds, and send 
e-mail to advise that data has been collected and is awaiting analysis. Or, if production batch throughput falls below 50 jobs per minute, activate a pager to the systems administrator on duty.

Obviously, **pmie** customization is required to produce meaningful filtering and actions in each production environment. The **pmieconf** tool provides a convenient customization method, allowing the user to 
generate parameterized **pmie** rules for some of the more common performance scenarios.

Basic pmie Usage
*****************

This section presents and explains some basic examples of **pmie** usage. The **pmie** tool accepts the common PCP command line arguments, as described in Chapter 3, :ref:`CommonConventionsandArguments`. In addition, **pmie** accepts the following command line arguments:

+-----------+----------------------------------------------------------------------------------------------------+
| **-d**    | Enables interactive debug mode.                                                                    |
+-----------+----------------------------------------------------------------------------------------------------+
| **-v**    | Verbose mode: expression values are displayed.                                                     |
+-----------+----------------------------------------------------------------------------------------------------+
| **-V**    | Verbose mode: annotated expression values are displayed.                                           |
+-----------+----------------------------------------------------------------------------------------------------+
| **-W**    | When-verbose mode: when a condition is true, the satisfying expression bindings are displayed.     |
+-----------+----------------------------------------------------------------------------------------------------+

One of the most basic invocations of this tool is this form::

 pmie filename

In this form, the expressions to be evaluated are read from *filename*. In the absence of a given *filename*, 
expressions are read from standard input, which may be your system keyboard.

pmie use of PCP services
=============================

Before you use **pmie**, it is strongly recommended that you familiarize yourself with the concepts from the Section 1.2, “:ref:`Conceptual Foundations`”. The discussion in this section serves as a very brief review of these concepts.

PCP makes available thousands of performance metrics that you can use when formulating expressions for **pmie** to evaluate. If you want to find out which metrics are currently available on your system, use this command::

 pminfo

Use the **pmie** command line arguments to find out more about a particular metric. In `Example 5.1. pmie with the -f Option`_, to fetch new metric values from host dove, you use the **-f** flag:

.. _Example 5.1. pmie with the -f Option:

**Example 5.1. pmie with the -f Option**

::
  
 pminfo -f -h dove disk.dev.total

This produces the following response::

 disk.dev.total
     inst [0 or "xscsi/pci00.01.0/target81/lun0/disc"] value 131233
     inst [4 or "xscsi/pci00.01.0/target82/lun0/disc"] value 4
     inst [8 or "xscsi/pci00.01.0/target83/lun0/disc"] value 4
     inst [12 or "xscsi/pci00.01.0/target84/lun0/disc"] value 4
     inst [16 or "xscsi/pci00.01.0/target85/lun0/disc"] value 4
     inst [18 or "xscsi/pci00.01.0/target86/lun0/disc"] value 4

This reveals that on the host **dove**, the metric **disk.dev.total** has six instances, one for each disk on the system.

Use the following command to request help text (specified with the **-T** flag) to provide more information about performance metrics::

 pminfo -T network.interface.in.packets

The metadata associated with a performance metric is used by **pmie** to determine how the value should be interpreted. You can examine the descriptor that encodes 
the metadata by using the **-d** flag for **pminfo**, as shown in `Example 5.2. pmie with the -d and -h Options`_ :

.. _Example 5.2. pmie with the -d and -h Options:

**Example 5.2. pmie with the -d and -h Options**

::

 pminfo -d -h somehost mem.util.cached kernel.percpu.cpu.user

In response, you see output similar to this::

 mem.util.cached
     Data Type: 64-bit unsigned int  InDom: PM_INDOM_NULL 0xffffffff
     Semantics: instant  Units: Kbyte

 kernel.percpu.cpu.user
     Data Type: 64-bit unsigned int  InDom: 60.0 0xf000000
     Semantics: counter  Units: millisec

.. note::
   A cumulative counter such as **kernel.percpu.cpu.user** is automatically converted by **pmie** into a rate (measured in events per second, or count/second), while 
   instantaneous values such as **mem.util.cached** are not subjected to rate conversion. Metrics with an instance domain (**InDom** in the **pminfo** output) of **PM_INDOM_NULL** 
   are singular and always produce one value per source. However, a metric like **kernel.percpu.cpu.user** has an instance domain, and may produce multiple values per 
   source (in this case, it is one value for each configured CPU).

⁠Simple pmie Usage
===================

`Example 5.3. pmie with the -v Option`_ directs the inference engine to evaluate and print values (specified with the **-v** flag) for a single performance metric (the 
simplest possible expression), in this case **disk.dev.total**, collected from the local PMCD:

.. _Example 5.3. pmie with the -v Option:

**Example 5.3. pmie with the -v Option**

::

 pmie -v
 iops = disk.dev.total;
 Ctrl+D
 iops:      ?      ?
 iops:   14.4      0
 iops:   25.9  0.112
 iops:   12.2      0
 iops:   12.3   64.1
 iops:  8.594  52.17
 iops:  2.001  71.64

On this system, there are two disk spindles, hence two values of the expression **iops** per sample. Notice that the values for the first sample are unknown 
(represented by the question marks [?] in the first line of output), because rates can be computed only when at least two samples are available. The subsequent 
samples are produced every ten seconds by default. The second sample reports that during the preceding ten seconds there was an average of 14.4 transfers per second 
on one disk and no transfers on the other disk.

Rates are computed using time stamps delivered by PMCD. Due to unavoidable inaccuracy in the actual sampling time (the sample interval is not exactly 10 seconds), 
you may see more decimal places in values than you expect. Notice, however, that these errors do not accumulate but cancel each other out over subsequent samples.

In `Example 5.3. pmie with the -v Option`_, the expression to be evaluated was entered using the keyboard, followed by the end-of-file character [**Ctrl+D**]. 
Usually, it is more convenient to enter expressions into a file (for example, **myrules**) and ask **pmie** to read the file. Use this command syntax::

 pmie -v myrules

Please refer to the **pmie(1)** man page for a complete description of **pmie** command line options.

⁠Complex pmie Examples
======================

This section illustrates more complex **pmie** expressions of the specification language. Section 5.3, “`Specification Language for pmie`_”, provides a complete 
description of the **pmie** specification language.

The following arithmetic expression computes the percentage of write operations over the total number of disk transfers.

::

 (disk.all.write / disk.all.total) * 100;

The **disk.all** metrics are singular, so this expression produces exactly one value per sample, independent of the number of disk devices.

.. note::

 If there is no disk activity, **disk.all.total** will be zero and **pmie** evaluates this expression to be not a number. When **-v** is used, any such values are displayed as question marks.

The following logical expression has the value **true** or **false** for each disk::

 disk.dev.total > 10 && 
 disk.dev.write > disk.dev.read;

The value is true if the number of writes exceeds the number of reads, and if there is significant disk activity (more than 10 transfers per second). 
`Example 5.4. Printed pmie Output`_ demonstrates a simple action:

.. _Example 5.4. Printed pmie Output:


**Example 5.4. Printed pmie Output**

::

 some_inst disk.dev.total > 60
           -> print "[%i] high disk i/o";

This prints a message to the standard output whenever the total number of transfers for some disk (**some_inst**) exceeds 60 transfers per second. The **%i** (instance) 
in the message is replaced with the name(s) of the disk(s) that caused the logical expression to be **true**.

Using **pmie** to evaluate the above expressions every 3 seconds, you see output similar to `Example 5.5. Labelled pmie Output`_. Notice the introduction of labels for each **pmie** expression.

.. _Example 5.5. Labelled pmie Output:

**Example 5.5. Labelled pmie Output**

::

 pmie -v -t 3sec
 pct_wrt = (disk.all.write / disk.all.total) * 100;
 busy_wrt = disk.dev.total > 10 &&
            disk.dev.write > disk.dev.read;
 busy = some_inst disk.dev.total > 60
            -> print "[%i] high disk i/o ";
 Ctrl+D
 pct_wrt:       ? 
 busy_wrt:      ?      ?
 busy:          ?
 
 pct_wrt:   18.43
 busy_wrt:  false  false
 busy:      false
 
 Mon Aug  5 14:56:08 2012: [disk2] high disk i/o
 pct_wrt:   10.83
 busy_wrt:  false  false
 busy:      true 
 
 pct_wrt:   19.85
 busy_wrt:   true  false
 busy:      false
 
 pct_wrt:       ?
 busy_wrt:  false  false
 busy:      false
 
 Mon Aug  5 14:56:17 2012: [disk1] high disk i/o [disk2] high disk i/o
 pct_wrt:   14.8
 busy_wrt:  false  false
 busy:   true

The first sample contains unknowns, since all expressions depend on computing rates. Also notice that the expression **pct_wrt** may have an undefined value whenever 
all disks are idle, as the denominator of the expression is zero. If one or more disks is busy, the expression **busy** is true, and the message from the **print** 
in the action part of the rule appears (before the **-v** values).

Specification Language for pmie
********************************

This section describes the complete syntax of the **pmie** specification language, as well as macro facilities and the issue of sampling and evaluation frequency. 
The reader with a preference for learning by example may choose to skip this section and go straight to the examples in Section 5.4, “`pmie Examples`_”.

Complex expressions are built up recursively from simple elements:

1. Performance metric values are obtained from PMCD for real-time sources, otherwise from PCP archive logs.
2. Metrics values may be combined using arithmetic operators to produce arithmetic expressions.
3. Arithmetic expressions may be compared using relational operators to produce logical expressions.
4. Logical expressions may be combined using Boolean operators, including powerful quantifiers.
5. Aggregation operators may be used to compute summary expressions, for either arithmetic or logical operands.
6. The final logical expression may be used to initiate a sequence of actions.

Basic pmie Syntax
==================

The **pmie** rule specification language supports a number of basic syntactic elements.

⁠Lexical Elements
-----------------

All **pmie** expressions are composed of the following lexical elements:

**Identifier**

Begins with an alphabetic character (either upper or lowercase), followed by zero or more letters, the numeric digits, and the special characters period (.) and 
underscore (_), as shown in the following example::

 x, disk.dev.total and my_stuff

As a special case, an arbitrary sequence of letters enclosed by apostrophes (') is also interpreted as an *identifier*; for example::

 'vms$slow_response'

**Keyword**

The aggregate operators, units, and predefined actions are represented by keywords; for example, **some_inst**, **print**, and **hour**.

Numeric constant
Any likely representation of a decimal integer or floating point number; for example, 124, 0.05, and -45.67

String constant

An arbitrary sequence of characters, enclosed by double quotation marks (**"x"**).

Within quotes of any sort, the backslash (\) may be used as an escape character as shown in the following example::

 "A \"gentle\" reminder"


Comments
---------

Comments may be embedded anywhere in the source, in either of these forms:

+--------------+---------------------------------------------------------------------------+
| /* text */   | Comment, optionally spanning multiple lines, with no nesting of comments. |
+--------------+---------------------------------------------------------------------------+
| // text      | Comment from here to the end of the line.                                 |
+--------------+---------------------------------------------------------------------------+

⁠Macros
-------

When they are fully specified, expressions in **pmie** tend to be verbose and repetitive. The use of macros can reduce repetition and improve readability and 
modularity. Any statement of the following form associates the macro name **identifier** with the given string constant.

:: 

 identifier = "string";

Any subsequent occurrence of the macro name **identifier** is replaced by the string most recently associated with a macro definition for **identifier**.

::

 $identifier 

For example, start with the following macro definition::

 disk = "disk.all";

You can then use the following syntax::

 pct_wrt = ($disk.write / $disk.total) * 100;

.. note::
   Macro expansion is performed before syntactic parsing; so macros may only be assigned constant string values.

Units
------

The inference engine converts all numeric values to canonical units (seconds for time, bytes for space, and events for count). To avoid surprises, you are encouraged to specify the units for numeric constants. If units are specified, they are checked for dimension compatibility against the metadata for the associated performance metrics.

The syntax for a **units** specification is a sequence of one or more of the following keywords separated by either a space or a slash (/), to denote per: **byte, KByte, MByte, GByte, TByte, nsec, nanosecond, usec, microsecond, msec, millisecond, sec, second, min, minute, hour, count, Kcount, Mcount, Gcount,** or **Tcount**. Plural forms are also accepted.

The following are examples of units usage::

 disk.dev.blktotal > 1 Mbyte / second; 
 mem.util.cached < 500 Kbyte;

.. note::
   If you do not specify the units for numeric constants, it is assumed that the constant is in the canonical units of seconds for time, bytes for space, and events for count, and the dimensionality of the constant is assumed to be correct. Thus, in the following expression, the **500** is interpreted as 500 bytes.

   ::

      mem.util.cached < 500

⁠Setting Evaluation Frequency
=============================

The identifier name **delta** is reserved to denote the interval of time between consecutive evaluations of one or more expressions. Set **delta** as follows::

 delta = number [units];

If present, **units** must be one of the time units described in the preceding section. If absent, **units** are assumed to be **seconds**. For example, the following 
expression has the effect that any subsequent expressions (up to the next expression that assigns a value to **delta**) are scheduled for evaluation at a fixed frequency, once every five minutes.

::

 delta = 5 min;

The default value for **delta** may be specified using the **-t** command line option; otherwise **delta** is initially set to be 10 seconds.

⁠pmie Metric Expressions
=========================

The performance metrics namespace (PMNS) provides a means of naming performance metrics, for example, **disk.dev.read**. PCP allows an application to retrieve one or more values for a performance metric from a designated source (a collector host running PMCD, or a set of PCP archive logs). To specify a single value for some performance metric requires the metric name to be associated with all three of the following:

1. A particular host (or source of metrics values) 
2. A particular instance (for metrics with multiple values)
3. A sample time

The permissible values for hosts are the range of valid hostnames as provided by Internet naming conventions.

The names for instances are provided by the Performance Metrics Domain Agents (PMDA) for the instance domain associated with the chosen performance metric.

The sample time specification is defined as the set of natural numbers 0, 1, 2, and so on. A number refers to one of a sequence of sampling events, from the current sample 0 to its predecessor 1, whose predecessor was 2, and so on. 
This scheme is illustrated by the time line shown in `Figure 5.1. Sampling Time Line`_.

.. _Figure 5.1. Sampling Time Line:

.. figure:: ../images/

    Figure 5.1. Sampling Time Line

Each sample point is assumed to be separated from its predecessor by a constant amount of real time, the **delta**. The most recent sample point is always zero. 
The value of **delta** may vary from one expression to the next, but is fixed for each expression; for more information on the sampling interval, see 
Section 5.3.2, “`Setting Evaluation Frequency`_”.

For **pmie**, a metrics expression is the name of a metric, optionally qualified by a host, instance and sample time specification. Special characters introduce 
the qualifiers: colon (**:**) for hosts, hash or pound sign (**#**) for instances, and at (**@**) for sample times. The following expression refers to the previous 
value (**@1**) of the counter for the disk read operations associated with the disk instance **#disk1** on the host **moomba**.

::

 disk.dev.read :moomba #disk1 @1

In fact, this expression defines a point in the three-dimensional (3D) parameter space of {**host**} x {**instance**} x {**sample time**} as shown in `Figure 5.2. Three-Dimensional Parameter Space`_.

.. _Figure 5.2. Three-Dimensional Parameter Space:
⁠
.. figure:: ../images/

    Figure 5.2. Three-Dimensional Parameter Space

A metric expression may also identify sets of values corresponding to one-, two-, or three-dimensional slices of this space, according to the following rules:

1. A metric expression consists of a PCP metric name, followed by optional host specifications, followed by optional instance specifications, and finally, optional sample time specifications.

2. A host specification consists of one or more host names, each prefixed by a colon (**:**). For example: ** /:indy /:far.away.domain.com /:localhost**

3. A missing host specification implies the default **pmie** source of metrics, as defined by a **-h** option on the command line, or the first named archive in an 
   **-a** option on the command line, or PMCD on the local host.

4. An instance specification consists of one or more instance names, each prefixed by a hash or pound (**#**) sign. For example: **#eth0 #eth2**

   Recall that you can discover the instance names for a particular metric, using the pminfo command. See Section 5.2.1, “`pmie use of PCP services`_”.

  Within the **pmie** grammar, an instance name is an identifier. If the instance name contains characters other than alphanumeric characters, enclose the instance name in single quotes; for example, **#'/boot' #'/usr'**

5. A missing instance specification implies all instances for the associated performance metric from each associated **pmie** source of metrics.

6. A sample time specification consists of either a single time or a range of times. A single time is represented as an at (**@**) followed by a natural number. 
   A range of times is an at (**@**), followed by a natural number, followed by two periods (**..**) followed by a second natural number. The ordering of the end 
   points in a range is immaterial. For example, **@0..9** specifies the last 10 sample times.

7. A missing sample time specification implies the most recent sample time.

The following metric expression refers to a three-dimensional set of values, with two hosts in one dimension, five sample times in another, and the number of instances 
in the third dimension being determined by the number of configured disk spindles on the two hosts.

::

 disk.dev.read :foo :bar @0..4

⁠pmie Rate Conversion
=====================

Many of the metrics delivered by PCP are cumulative counters. Consider the following metric::

 disk.all.total

A single value for this metric tells you only that a certain number of disk I/O operations have occurred since boot time, and that information may be invalid if the 
counter has exceeded its 32-bit range and wrapped. You need at least two values, sampled at known times, to compute the recent rate at which the I/O operations are 
being executed. The required syntax would be this::

 (disk.all.total @0 - disk.all.total @1) / delta

The accuracy of **delta** as a measure of actual inter-sample delay is an issue. **pmie** requests samples, at intervals of approximately **delta**, while the results 
exported from PMCD are time stamped with the high-resolution system clock time when the samples were extracted. For these reasons, a built-in and implicit rate 
conversion using accurate time stamps is provided by **pmie** for performance metrics that have counter semantics. For example, the following expression is 
unconditionally converted to a rate by pmie.

::

 disk.all.total

⁠pmie Arithmetic Expressions
============================

Within **pmie**, simple arithmetic expressions are constructed from metrics expressions (see Section 5.3.3, “`pmie Metric Expressions`_”) and numeric constants, 
using all of the arithmetic operators and precedence rules of the C programming language.

All **pmie** arithmetic is performed in double precision.

Section 5.3.8, “`pmie Intrinsic Operators`_”, describes additional operators that may be used for aggregate operations to reduce the dimensionality of an arithmetic expression.

⁠pmie Logical Expressions
=========================

A number of logical expression types are supported:

* Logical constants
* Relational expressions
* Boolean expressions
* Quantification operators

Logical Constants
------------------

Like in the C programming language, **pmie** interprets an arithmetic value of zero to be false, and all other arithmetic values are considered true.

⁠Relational Expressions
-----------------------

Relational expressions are the simplest form of logical expression, in which values may be derived from arithmetic expressions using ***pmie** relational operators. 
For example, the following is a relational expression that is true or false, depending on the aggregate total of disk read operations per second being greater than 50.

::

 disk.all.read > 50 count/sec

All of the relational logical operators and precedence rules of the C programming language are supported in **pmie**.

As described in Section 5.3.3, “`pmie Metric Expressions`_”, arithmetic expressions in **pmie** may assume set values. The relational operators are also required to 
take constant, singleton, and set-valued expressions as arguments. The result has the same dimensionality as the operands. Suppose the rule in `Example 5.6. Relational Expressions`_ is given:

.. _Example 5.6. Relational Expressions:

**Example 5.6. Relational Expressions**

::
 
 hosts = ":gonzo";
 intfs = "#eth0 #eth2";
 all_intf = network.interface.in.packets
                $hosts $intfs @0..2 > 300 count/sec;

Then the execution of **pmie** may proceed as follows:

::

 pmie -V uag.11
 all_intf: 
        gonzo: [eth0]      ?      ?      ? 
        gonzo: [eth2]      ?      ?      ?
 all_intf:
        gonzo: [eth0]  false      ?      ?
        gonzo: [eth2]  false      ?      ?
 all_intf:
        gonzo: [eth0]   true  false      ?
        gonzo: [eth2]  false  false      ?
 all_intf:
        gonzo: [eth0]   true   true  false
        gonzo: [eth2]  false  false  false

At each sample, the relational operator greater than (>) produces six truth values for the cross-product of the **instance** and **sample time** dimensions.

Section 5.3.6.4, “`Quantification Operators`_”, describes additional logical operators that may be used to reduce the dimensionality of a relational expression.

⁠Boolean Expressions
--------------------

The regular Boolean operators from the C programming language are supported: conjunction (**&&**), disjunction (**||**) and negation (**!**).

As with the relational operators, the Boolean operators accommodate set-valued operands, and set-valued results.

Quantification Operators
-------------------------

Boolean and relational operators may accept set-valued operands and produce set-valued results. In many cases, rules that are appropriate for performance management 
require a set of truth values to be reduced along one or more of the dimensions of hosts, instances, and sample times described in Section 5.3.3, “`pmie Metric Expressions`_”. 
The **pmie** quantification operators perform this function.

Each quantification operator takes a one-, two-, or three-dimension set of truth values as an operand, and reduces it to a set of smaller dimension, by quantification 
along a single dimension. For example, suppose the expression in the previous example is simplified and prefixed by **some_sample**, to produce the following expression::

 intfs = "#eth0 #eth2"; 
 all_intf = some_sample network.interface.in.packets
                      $intfs @0..2 > 300 count/sec;

Then the expression result is reduced from six values to two (one per interface instance), such that the result for a particular instance will be false unless the 
relational expression for the same interface instance is true for at least one of the preceding three sample times.

There are existential, universal, and percentile quantification operators in each of the *host, instance*, and *sample time* dimensions to produce the nine operators as follows:

+--------------------+----------------------------------------------------------------------------------------------------+
| some_host          | True if the expression is true for at least one host for the same instance and sample time.        |
+--------------------+----------------------------------------------------------------------------------------------------+
| all_host           | True if the expression is true for every host for the same instance and sample time.               |
+--------------------+----------------------------------------------------------------------------------------------------+
| N%_host            | True if the expression is true for at least N% of the hosts for the same instance and sample time. |
+--------------------+----------------------------------------------------------------------------------------------------+
| some_inst          | True if the expression is true for at least one instance for the same host and sample time.        |
+--------------------+----------------------------------------------------------------------------------------------------+
| all_instance       | True if the expression is true for every instance for the same host and sample time.               |
+--------------------+----------------------------------------------------------------------------------------------------+
| N%_instance        | True if the expression is true for at least N% of the instances for the same host and sample time. |
+--------------------+----------------------------------------------------------------------------------------------------+
| some_sample time   | True if the expression is true for at least one sample time for the same host and instance.        |
+--------------------+----------------------------------------------------------------------------------------------------+
| all_sample time    | True if the expression is true for every sample time for the same host and instance.               |
+--------------------+----------------------------------------------------------------------------------------------------+
| N%_sample time     | True if the expression is true for at least N% of the sample times for the same host and instance. |
+--------------------+----------------------------------------------------------------------------------------------------+

These operators may be nested. For example, the following expression answers the question: “Are all hosts experiencing at least 20% of their disks busy either reading or writing?”

::

 Servers = ":moomba :babylon";
 all_host ( 
     20%_inst disk.dev.read $Servers > 40 || 
     20%_inst disk.dev.write $Servers > 40
 );

The following expression uses different syntax to encode the same semantics::

 all_host (
     20%_inst (
         disk.dev.read $Servers > 40 ||
         disk.dev.write $Servers > 40
     )
 );

.. note::
   To avoid confusion over precedence and scope for the quantification operators, use explicit parentheses.

Two additional quantification operators are available for the instance dimension only, namely **match_inst** and **nomatch_inst**, that take a regular expression and a 
boolean expression. The result is the boolean AND of the expression and the result of matching (or not matching) the associated instance name against the regular expression.

For example, this rule evaluates error rates on various 10BaseT Ethernet network interfaces (such as ecN, ethN, or efN):

::

 some_inst
         match_inst "^(ec|eth|ef)"
                 network.interface.total.errors > 10 count/sec
 -> syslog "Ethernet errors:" " %i"
 
pmie Rule Expressions
======================

Rule expressions for **pmie** have the following syntax::

 lexpr -> actions ;

The semantics are as follows:

* If the logical expression **lexpr** evaluates **true**, then perform the *actions* that follow. Otherwise, do not perform the *actions*.
* It is required that **lexpr** has a singular truth value. Aggregation and quantification operators must have been applied to reduce multiple truth values to a single value.
* When executed, an *action* completes with a success/failure status.
* One or more *actions* may appear; consecutive *actions* are separated by operators that control the execution of subsequent *actions*, as follows:
   
   * *action-1* **&** : Always execute subsequent actions (serial execution).
   * *action-1* **|** : If *action-1* fails, execute subsequent actions, otherwise skip the subsequent actions (alternation).

An *action* is composed of a keyword to identify the action method, an optional *time* specification, and one or more arguments.

A *time* specification uses the same syntax as a valid time interval that may be assigned to **delta**, as described in Section 5.3.2, "`Setting Evaluation Frequency`_ ”. 
If the *action* is executed and the *time* specification is present, **pmie** will suppress any subsequent execution of this *action* until the wall clock time has advanced by *time*.

The arguments are passed directly to the action method.

The following action methods are provided:

**shell**

The single argument is passed to the shell for execution. This *action* is implemented using **system** in the background. The *action* does not wait for the system call to return, and succeeds unless the fork fails.

**alarm**

A notifier containing a time stamp, a single *argument* as a message, and a **Cancel** button is posted on the current display screen (as identified by the **DISPLAY** 
environment variable). Each alarm *action* first checks if its notifier is already active. If there is an identical active notifier, a duplicate notifier is not posted. 
The action succeeds unless the fork fails.

**syslog**

A message is written into the system log. If the first word of the first argument is **-p**, the second word is interpreted as the priority (see the **syslog(3)** man page); the message tag is **pcp-pmie**. 
The remaining argument is the message to be written to the system log. This action always succeeds.

**print**