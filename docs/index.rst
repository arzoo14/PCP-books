.. pcp documentation master file, created by
   sphinx-quickstart on Wed Sep 16 15:12:41 2020.

Performance Co-Pilot
####################

`Performance Co-Pilot (PCP) <https://pcp.io/>`_ provides a framework and services to support system-level performance monitoring and management. It presents a unifying 
abstraction for all of the performance data in a system, and many tools for interrogating, retrieving and processing that data.

PCP is a feature-rich, mature, extensible, cross-platform toolkit supporting both live and retrospective analysis. The distributed PCP architecture 
makes it especially useful for those seeking centralized monitoring of distributed processing.

**Table of Contents**

* :doc:`HowTos/installation/index`
* :doc:`HowDoI/HowDoIGuide`
* :doc:`UAG/AboutUserAdministratorsGuide`
* :doc:`PG/AboutProgrammersGuide`
* :doc:`HowTos/scaling/index`
* `REST API Specification <api/>`_

.. toctree::
   :caption: Guides
   :hidden:
   
   HowTos/installation/index
   HowDoI/HowDoIGuide
   UAG/AboutUserAdministratorsGuide
   PG/AboutProgrammersGuide
   HowTos/scaling/index
   REST API Specification <https://pcp.readthedocs.io/en/latest/api/>

.. toctree::
   :caption: How Do I Guide
   :numbered:
   :hidden:

   HowDoI/ListAvailableMetrics
   HowDoI/AddNewMetrics
   HowDoI/RecordMetricsOnLocalSystem
   HowDoI/RecordMetricsFromRemoteSystem
   HowDoI/GraphPerformanceMetric
   HowDoI/AutomateProblemDetection
   HowDoI/SetupAutomatedRules
   HowDoI/RecordHistoricalValues
   HowDoI/ExportMetricValues
   HowDoI/UseCharts
   HowDoI/LoggingBasics
   HowDoI/AutomatedReasoningBasics
   HowDoI/ConfigureAutomatedReasoning
   HowDoI/AnalyzeLinuxContainers
   HowDoI/SecureConnections
   HowDoI/SecureClientConnections
   HowDoI/AuthenticatedConnections
   HowDoI/ImportData
   HowDoI/Use3DViews


.. toctree::
   :caption: User's and Administrator's Guide
   :numbered:
   :hidden:

   UAG/IntroductionToPcp
   UAG/InstallingAndConfiguringPcp
   UAG/CommonConventionsAndArguments
   UAG/MonitoringSystemPerformance
   UAG/PerformanceMetricsInferenceEngine
   UAG/ArchiveLogging
   UAG/PcpDeploymentStrategies
   UAG/CustomizingAndExtendingPcpServices
   UAG/TimeSeriesQuerying


.. toctree::
   :caption: Programmer's Guide
   :numbered:
   :hidden:

   PG/ProgrammingPcp
   PG/WritingPMDA
   PG/PMAPI
   PG/InstrumentingApplications
