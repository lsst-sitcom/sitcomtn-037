:tocdepth: 2

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

.. _SITCOM-173: https://jira.lsstcorp.org/browse/SITCOM-173
.. _SITCOM-174: https://jira.lsstcorp.org/browse/SITCOM-174
.. _SITCOM-180: https://jira.lsstcorp.org/browse/SITCOM-180
.. _Prompt Processing: https://dmtn-219.lsst.io/
.. _charge: https://sitcomtn-030.lsst.io/
.. _FAFF report: https://sitcomtn-025.lsst.io/

Abstract
========

The second `charge`_ to the First-Look Analysis and Feedback Functionality Breakout Group details questions regarding the displays and functionalities required to perform continuous nightly commissioning activities.
The focus is on deriving the required functionalities to perform inspection and analyses within approximately 24-hours of taking the data.
This document details the answers to the charge questions and other findings.

Executive Summary
=================

The second First-Look Analysis and Feedback Functionality (FAFF) report details the computational tasks and interactions with resultant data products that are needed to support the full 24-hour cycle of sustained observatory operations.
Starting from a set of use-cases derived from expected commissioning needs and experience with other observatories, we identified two major compute-intensive tasks that are required for near-realtime evaluation of data quality to support nighttime decision making.
First, a **Rapid Analysis Framework** is needed to run single-frame processing (SFP) through the source detection and measurement steps on bright stars in order to make basic measurements of delivered image quality and optical throughput.
Second, a **camera image visualization tool** is needed to rapidly display full focal plane images and zoom in to inspect pixel-level details.
We recommend that compute resources to support both of these tasks be located on the summit so that observers have near-realtime data quality information even in the event of a network outage.
Basic camera health diagnostics and image display capabilities will run on the Camera Diagnostic Cluster located at the summit.
Rapid analysis would run on the commissioning (Antu) cluster that is currently in the process of being relocated to the summit.
Making these compute resources available for use at the summit, especially Rapid Analysis, is the highest priority recommendation of the FAFF report.

Once compute resources are available at the summit, the next priorities are to deploy the Rapid Analysis framework, set up the Camera Diagnostic Cluster to perform low-level camera health checks, stand up databases to record the time series of telemetry together with metric values produced by Rapid Analysis and camera health diagnostics (e.g., Sasquatch), and develop the framework for generating alerts that metric values and reduced images are available.
With this infrastructure in place, we will begin using the tools together to generate, record, alert, and then display metric values.
From this stage, we will expand capabilities to produce a variety of displays of time series of metric values and corresponding telemetry data (Chronograf), scalar fields (Bokeh apps), visualizing full focal plane images (Camera Visualization Tool), as well as generating alarms based on conditions and automatically producing more detailed artifacts (Catcher).
Emphasis should initially be on exercising the interfaces between these tools and then expanding the capabilities of individual tools.

While drafting the FAFF report, we explored several demonstrations of the functionality described above, but implementation of an end-to-end system is beyond the scope of group.
Many of the tasks above can be pursued in parallel, but draw upon a specific small set of developers, and thus careful coordination and management is needed.
More individuals will be able to effectively contribute as continued effort is put into developing templates and extensible frameworks.
In addition, we identified several capabilities, such as logging tools and a tabular summary of recent exposures with linked diagnostic information, that would already be in regular use with AuxTel observations but are not currently owned by any person/group.
Most of the planned tooling for visualizing data quality information should be sufficiently straightforward to not require dedicated training for users.

Introduction
============

This report addresses the second `charge`_ to the First-Look Analysis and Feedback Functionality (FAFF) Breakout Group.
The charge was developed based on the findings during the Missing Functionality Workshop that took place February 2nd and 3rd, 2022.
The charge addresses three specific Jira tickets developed from that meeting: `SITCOM-174`_, `SITCOM-173`_, and `SITCOM-180`_.
These are referenced throughout the report.
This working group is focused on what is needed to perform inspection and analyses required within the first 24 hours of taking the data and is not aiming to address all visualization requirements from the `Observatory System Specifications Document <https://ls.st/lse-30>`_.

Much of this report builds off the findings and recommendations of the first `FAFF report`_.
It is expected that the readers are already familiar with the findings and recommendations.

The remainder of this report is structured into sections that address each of the charge questions individually.
The relevant charge question has been copied into each section for ease-of-readability.
The report concludes with a section on findings that are pertinent to the commissioning team but fall outside the scope of the original `charge`_ .
It is recommended that these issues get addressed by either a follow-on charge or another mechanism.

Rapid Analysis Capability
-------------------------

Most of the envisioned FAFF functionality requires data products based on both observatory telemetry and camera images to be available for immediate use at the summit in order to display this information and thereby inform nighttime operations.
While developing use cases, we recorded the input data products and associated timescales for their use, and then compiled these into a consolidated list to better understand the set of required generation mechanisms.

We find that multiple FAFF use cases require a `Rapid Analysis <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_ capability that includes automated SFP of camera images through instrument signature removal and source detection on the 30-60 second timescale to enable the creation of various data quality metrics and data visualizations.
The Confluence page linked above describes the needed data products and timescales for their creation, as requested in `charge`_  question 2.
The use cases developed for charge question 1 assume that the data products from this initial processing step are available.

`Prompt Processing`_ is a framework for continuously processing the stream of images coming off the telescope, for example, to execute Science Pipeline payloads for Alert Production including Solar System Processing.
Prompt Processing runs at the USDF.
Needs for near-realtime data quality assessment could be partially addressed by a suitable payload executed with the Prompt Processing framework at USDF, provided that results are made available to the summit.
A dedicated Rapid Analysis capability at the summit would ensure that on-the-fly diagnostic information is available to observers even if connection to the base is lost, as discussed in the :ref:`Antu at Summit <antu_at_summit>` scenario.

Responses to Charge Questions and Deliverables
==============================================

The use-cases referenced throughout the document are all found on the `dedicated confluence page <https://confluence.lsstcorp.org/display/LSSTCOM/Use-Cases>`_ and are further described below.

.. _Deliverable 1:

Deliverable 1: Use-Cases
------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   1. (`SITCOM-174`_, `SITCOM-173`_, `SITCOM-180`_) A series of use-cases where image data analysis is required on short timescales.
      A reduced set of use-cases should be created as a regular reference throughout the charge.
      A set of required turn-around time(s) should be defined and assigned to each case where applicable.

      - Use-cases should be complete, including which inputs are required and from where they will originate (e.g. SAL Script, EFD, LFA, external source), desired manipulations, logic-based operations/calculations, and if/how the desired artifacts are presented to the user (e.g. display images and/or graphs).


Numerous use cases were developed to capture the needed functionalities and assist in developing a common understanding of what is expected in each scenario.
Each of the use cases follow a standardized `template <https://confluence.lsstcorp.org/display/LSSTCOM/On-the-fly+Analysis+Use-Case+Template>`_ which differs slightly from that which was used in the first FAFF charge.

The remaining use-cases for FAFF2 can be found on the FAFF use-cases page `on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Use-Cases>`_ and are referenced throughout the remainder of this report.

Daytime Calibration
^^^^^^^^^^^^^^^^^^^

.. warning::

   This section is not yet completed.


During the course of the working group, the example of daytime calibration was raised repeatedly, specifically in regards to how calibration data products are generated and what is expected of the observing specialist.
The aspect pertaining specifically to the FAFF charge is what the observer is required to look at during the process, including both images and/or alarms.
The details of how Daytime Calibration is performed is being documented in `DMTN-222 <https://DMTN-222.lsst.io>`_ and will not be repeated as a new use-case.

In short, a SAL script is launched by the observer to acquire a daytime set of calibrations.
This SAL script launches an OCPS-based processing of the images, but the ScriptQueue does not block on the processing awaiting the final analysis.
Currently, if the process fails then no alert is generated automatically.
However, as will be discussed in the following sections, a Watcher [#]_ alarm will be setup to listen and alert users (via LOVE) in the event of a catastrophic failure in the analysis which the observer could do something about (e.g. the shutter did not open and the flats have no signal).
How the observer responds to the alert is currently being discussed.
Presumably, this will use a parameterized notebook that will allow an observer to better understand the issue.
Any viewing of the raw frames themselves will utilize the Camera Visualization Tool.

In the case where a more complex issue arises (e.g., a 2% increase in bad pixels is observed), this is addressed by the calibration team offsite and is not immediately reported to the summit team.
When the calibrations used on the summit need to be updated, this is the role of the calibration scientist and is not the responsibility of the observer.
Furthermore, this cadence is expected to be slow (months) and is therefore outside the scope of this charge.

.. [#] The `Watcher CSC <https://ts-watcher.lsst.io/>`_ is provided a list of "rules" that it ensures the system is always obeying. If a rule is violated, such as a temperature going out of specification, the an alert or alarm is issued to the observer via the LOVE interface. The alarm stays in place until the rule is no longer violated and the original alert has been acknowledged. The Watcher is not able to perform analyses and only evaluates simple conditions.

.. _Deliverable 2:

Deliverable 2: Rapid Analysis Calculated Metrics
------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.


   2. (`SITCOM-180`_, `SITCOM-173`_) Define which metrics, analyses and artifacts must be calculated and on what timescale they must be evaluated and reported to support commissioning/operations.

      This is to evaluate if a "rapid processing" of data is required, what specific calculations are required.
      This list should include the relevant camera specific calculations (which are currently performed by the EO testing data reduction).
      This is expected to inform the answer to the next charge task.


It is important to note that the charge question above refers to "rapid processing."
We intentionally avoid the use of this term and have adopted the phrasing, "Rapid Analysis" instead.
This is to avoid any potential confusion with Prompt Processing, which is a already defined set of functionalities.

Numerous calculations are required to evaluate camera and system health and performance on rapid timescales.
The data products discussed in this section are limited to scalars and/or arrays and do *not* include diagnostic plots and/or figures (visualization use cases are discussed separately).
The large majority of data products needed on rapid timescales are produced as part of the Science Pipelines single-frame-processing (SFP) framework.
A small number of additional values are also required, but can be quickly derived from the SFP results.
The calculated values from Rapid Analysis are not to produce data products that are critical to commissioning (`FAFF-REQ-0053`_), however, it is expected that observatory functionality is reduced if an outage were to occur.
This implies that the Rapid Analysis is not required to run at the summit, although if would be preferable to do so.
The output from the Rapid Analysis will need to go into a database.
Details of this are database are discussed in `Deliverable 3`_.
The output will also have to be made available to the control network such that observers can be alerted if calculated metrics are producing results that are deemed worrisome.
The original framework to perform this duty is the Telemetry Interface, described in :ref:`LSE-72`, which is designed to feed metrics from Prompt Processing pipelines back to the summit.
This document is out-of-date, however, either this or an analogous framework is required to perform the same purpose, such that the Watcher CSC can monitor for troubling events and alert the observer.

Based on the committee's experience commissioning previous telescopes, instruments and surveys, three different timescales for data interaction were identified as being critical to successful commissioning, each of which are discussed in the following subsections.
The data products for the rapid timescales (<30 and 60 seconds) are described in the Outputs section of the `Rapid Analysis Use-case on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_.

<30 seconds
^^^^^^^^^^^
This is the timescale where the data feedback must be made available quickly because it could potentially influence the next activity, configuration, or exposure.
Examples of required functionality at this timescale include displaying of images and evaluation and display of fundamental health metrics.
In the case of performing engineering tasks where corrections or instrument setups are being modified, it is useful to know if the changes impacted the next image as anticipated.
An example of this would be looking at PSF changes as a function of mirror shape or AOS configuration.

The camera commissioning cluster is unique as it is the first significant computing infrastructure to have access to the pixel data.
This is where the Camera Visualization Tool (CVT) is to be run such that users can see the images with the lowest possible latency.
It is also where the camera system conducts low-level measurements to determine camera health, such as median and standard deviation of the overscan regions.
This is then used to help inform the camera health displays, as discussed in the `specific use-case <https://confluence.lsstcorp.org/display/LSSTCOM/Camera+health+check>`_.
Further details regarding use of the commissioning cluster are discussed in `Deliverable 5`_.

The Rapid Analysis pipeline is to be run SFP on the Antu servers (the commissioning cluster), where more compute is available and the hardware consists of generic and more easily managed servers.
There are values in the SFP pipeline that are more pertinent to have on shorter timescales, such as the PSF shape.
These values have been identified in the `Rapid Analysis Use-case <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_ and if it is possible to output them prior to others it would help increase operational efficiency.

~60 seconds
^^^^^^^^^^^
This timescale is useful when examining trending or slowly varying effects, particularly for metrics like image quality or transparency.
It is a timescale where people are closely watching, but not necessarily immediately reacting.
The addition of this category was to provide flexibility in implementation as it may be such that the prioritization of metrics can be performed which may provide a useful free parameter during the implementation phase.
However, it is imperative that the Rapid Analysis framework be able to keep up with the rate of images being acquired; where that rate is governed by the survey strategy visit duration (`FAFF-REQ-0051`_).
In the case of taking two 15 second snaps, it is expected that the analysis would be done on the combined images.

Again, the data products for the 60 second timescales are described in the Outputs section of the `Rapid Analysis Use-case <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_.


12-24 hours
^^^^^^^^^^^
This timescale is important for more general commissioning activities and performance assessment that could impact observations taken in the next or subsequent nights.
Over this timescale, a full SFP pipeline needs to be run (`FAFF-REQ-0052`_).
This must include the additional values that are calculated in the Rapid Analysis Framework, which will need to be added to the SFP pipeline.
Re-calculation of these values enables a more detailed and higher-confidence data quality evaluation to be performed, including correlation with telemetry, environmental conditions, and previous conditions and/or observations.
It also allows the teams to begin determining which subsets of data should be used to construct coadds/templates, begin SV analyses, and ultimately maximize the number of human brain cycles looking at the data.
It is fully expected that this dataset will be superseded by a subsequent DRP campaign to enforce that all the data is processed in a homogeneous way with best performing configuration of the science pipelines.

It is not required that the full SFP processing be done in Chile, in fact, it is *preferable* to perform this processing at the USDF as many of the science verification tasks are planned to be performed there as well.
It also ensures that a minimum number of users are connecting to Chile to perform their analysis.
This is especially important if connections would be required to the summit instance.

Lastly, the results of this analysis do not need to be forwarded back to the summit control system.

Potential Paths for Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Rapid Analysis framework relies heavily on SFP, and therefore is a compatible payload with both the DRP and the Alert Production Pipelines.
The ultimate implementation decisions are outside the FAFF scope, however, because of the speed requirements, which will necessitate the pre-loading of expected image properties into memory (e.g. catalogues), it is expected that the path of least resistance would be to work with the Alert Production team in the development of Rapid Analysis.

Another aspect which may impact implementation is that Rapid Analysis only needs to run once per frame.
Even upon a failure to produce one of the parameters, or the publishing of an incorrect result, the system will not be rerun and therefore the database containing the results does not need to support versioning or relationships to previous results.

A re-occurring concern has been whether or not the Antu servers can support the Rapid Analysis framework.
FAFF has worked with Rubin project members to create a preliminary analysis of the compute required to run Rapid Analysis and found the following:

.. at with ~250 cores (1 per detector and a handful for overhead), combined with some attention paid to code performance enhancements, we expect that in terms of processing, keeping up with a 30s image cadence is very feasible.


- ~4 cores per CCD are required to perform the data processing
- Using the full 189 CCDs also requires 756 cores which is nearly the entire Antu cluster (784 cores)
- To support required data Input/Output (I/O), a cluster would ideally have a small number of cores per node, then spread the data out across multiple disks.
  Antu has a high core-to-node ratio, and is therefore likely unable to run Rapid Analysis for the entire array at a ~30s cadence.


At the moment, it is unclear if the computing infrastructure could be augmented to support full-frame on-the-fly processing in Chile.
If not, then the remaining option is to reduce the number of CCDs that get processed.
DECam encountered the same constraints and invoked a pipeline that supports different configurations that specify various patterns of sensors to reduce.
For example, pointing tests used just the central portion of the focal plane.
A list of possible focal plane configurations should be created; we have already reached out to the AOS[#]_ and Science Verification[#]_ groups for suggestions.
It is recommended that Rubin adopt a similar architecture as it is not expected that any summit-based Rapid Analysis image quality metrics would require the full array.
Especially since the camera diagnostic cluster handles the low-level health checks for all sensors, as is discussed in `Deliverable 5`_.

The University of Washington group is now investigating the SFP performance enhancements.
Scaling the experience gained with LATISS, it is expected that a 30s image cadence is feasible and the primary speed limitation will be the I/O constraints.

.. [#] The AOS group has already communicated that a checkerboard pattern for the focal plane, while omitting the 8 outermost sensors which are highly vignetted, is satisfactory to accomplish their analysis requirements.

.. [#] The Science Verification group has indicated that full-frame on-the-fly processing is not required, so long as full frame processing occurs at the USDF within 24-hours.

.. _analysis_tools_overview:

analysis_tools
^^^^^^^^^^^^^^

Several `basic per-detector data quality statistics <https://confluence.lsstcorp.org/display/LSSTCOM/Science+performance+metrics+to+support+nightly+operations>`_ are generated during SFP and persisted in the Butler repository.
These basic quantities can be supplemented by more detailed data quality diagnostics produced by other Science Pipeline components.

The recently released analysis_tools python package is a refactor of the faro and analysis_drp packages that provides both metric and plot generation functionality.
The package includes a set of analysis modules that can be run as Tasks within a data reduction pipeline, as part of a separate afterburner pipeline, or imported and executed within a standalone in a script/notebook.
The new package more fully leverages middleware capabilities, e.g., high configurability and efficient grouping of analyses into quanta with a smaller number of output files.
Metric values and plots are persisted alongside the input data products in the same Butler repository.
Importantly, analysis_tools adds the ability to easily reconstitute input data products along with the configuration that was used to generate a given metric/plot to enable interactive drill-down analyses.
The package adopts a modular design to encourage re-using code for metric calculation and visualization.
Currently implemented analyses include metrics and plots that run on per-visit source tables, per-tract object tables, per-tract associated sources, and difference image analysis source and object tables.

analysis_tools was added to main distribution of Science Pipelines (lsst_distrib) in August 2022.
The package now includes multiple example metrics and plots for single-visit, coadd, and DIA data quality assessment.

For examples, see the `tutorial notebook <https://github.com/lsst-dm/analysis_tools_examples>`_ shown at the Rubin PCW 2022.

.. _Deliverable 3:

Deliverable 3: Interacting with Rapid Analysis Data and Metrics
---------------------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   1. (`SITCOM-174`_, `SITCOM-173`_) Define how users will interact with each aspect of the previously listed metrics, analyses and artifacts; classify them indicating where can could calculated.

      This includes tasks defined for the catcher, OCPS jobs, AuxTel/ComCam/LSSTCam processing, and the rendez-vous of data from multiple sources (DIMM, all-sky etc).

.. warning::

   This section is not yet completed.

Simple scalar metrics (e.g., DIMM measured seeing) are easily visualized with tools like Chronograf, and are not addressed here.
They can be considered a subset of the scalar fields case below.
This section considers the case of scalar fields, where the same metric is plotted for multiple data origins.
A straightforward example to consider is a metric as a function of detector and/or amplifier on the focal plane.

The use of scalar fields will be displayed using various visualization tools and/or frameworks.
Examples include:

- Camera visualization health tool(s) which will display metrics for each amp/sensor.
- Scheduler Troubleshooting
- Extended functionality of the CVT (but better captured in the section, `Deliverable 6`_)
- Bokeh Apps embedded into the LOVE framework
- Webpages (TBD how this would be used, Noteburst+Times Square is an option)
- Trending plots (see also `Deliverable 4`_ for discussion of scalar fields as a function of a 3rd axis)

It is useful to group into aggregated (binned) and non-aggregated (unbinned) metrics.

- Binned: aggregated values that are pre-computed on a specified spatial scale (e.g. an amplifier, detector, raft, or telescope position), where the scaling could potentially modified. Depending on the case, a slider could be present to adjust the scaling on-the-fly
- Unbinned: Value per source (e.g. photometry measurement at each previous visit).

After significant discussion, it was determined that operations on the mountain and within the first ~24 hours of taking data, it is sufficient to deal with *only* aggregated data.
However, multiple forms of aggregation need to be supported (per amp, per detector, per raft, per HEALPix, sq degree etc.)
Analysis of unbinned data is clearly needed for pipeline data quality analyses, however, this is not something that will be diagnosed during the night by the summit crew.


Databases
^^^^^^^^^

.. warning::

   This section is not yet completed and only reports the current status.

Data from the observatory will come from numerous sources and efforts should be made to minimize the number of individual databases; both for maintenance and ease-of-use reasons.
Whereas much of the data coming off the summit is time based and therefore goes into a time-based database (the EFD), other aspects of the system are image based, such as what will be produced by Rapid Analysis and the parts of the camera system.
The implementation of various project databases is currently being discussed and documented in a number of tech notes[*]_ however, the capabilities and functionalities required by the commissioning team has not been explicitly described.

.. [*] For further details, consult the following technotes, which are in various states of being written: `Sasquatch <https://sqr058.lsst.io>`_, the `Butler <https://dmtn-204.lsst.io>`,  database support for `campaigns <https://dmtn-220.lsst.io/>`_, as well as the `consolidated database <https://dmtn-227.lsst.io/>`_.

FAFF is assembling a series of use-cases, specifically descriptions of database queries, that will identify the commissioning-specific functionalities required by the project databases.
This content is currently hosted on `a confluence page <https://confluence.lsstcorp.org/display/LSSTCOM/Use+cases+for+commissioning+databases>`_, but the pertinent content will be merged to this report and/or the use-cases described as part of `Deliverable 1`_.

Independent of the work describe above, early discussions have already yielded the following requirements on the database infrastructure, with more to come as the work progresses:

-  Users require a framework/method that manages the point(s) of access, analogous to the EFD Client (`FAFF-REQ-0055`_).
   Ideally, users will have the impression all queries are going to a single database, despite what is actually happening on the back-end(s).
- The database must be available and rapidly synced to at all major data facilities (`FAFF-REQ-0055`_), analogous to what is done for the summit EFD.
- Summit tooling, including the Scheduler, must have immediate access to the database (`FAFF-REQ-0056`_).

..
   Plot Visualization
   ^^^^^^^^^^^^^^^^^^^
   Use and expansion of the plot visualization tool.
   Also explain the current use of RubinTV

.. _Deliverable 4:

Deliverable 4: Required Non-Scalar Metrics
------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  4. (`SITCOM-180`_) Provide a list of required non-scalar metrics are required and cannot be computed with analysis_tools.
     Suggest a mechanism (work flow) to perform the measurement, document the finding, evaluate any trend (if applicable), then present it to the stakeholders.


.. related to https://confluence.lsstcorp.org/display/LSSTCOM/Displaying+scalar+fields+as+a+function+of+other+parameters

This charge question covers the issue of calculating and displaying the trending of scalar fields.
Scalar fields are single value metrics, but calculated per spatial element, as described in `Deliverable 3`_.
This charge question deals with adding a third dimension to the scalar field, then calculating and displaying this data to the user.
For example, this could be displaying the PSF width for each detector as a function of elevation, or sky transparency as a function of time.
As discussed above, both of these examples deal with aggregated (binned) data.

Currently, `analysis_tools`_ computes a bundle of single-valued (scalar) metrics on individual visits.
With small modifications, the package could persist arrays of metric values (e.g., per detector or finer granularity) that could be aggregated and visualized in flexible ways by downstream tooling.
The package already produces and persists static plots for displaying scalar fields in focal plane coordinates.

After analyzing the use-cases, including hypotheticals not detailed in the report, it was decided that there is not a use-case where we are unable represent a scalar field with respect to a third axis (e.g. time, elevation etc) as a single valued metric (e.g. a mean, or standard deviation), so long as the desired aggregation is supported.
Taking the examples discussed above, one would reduce the scalar field to a number of scalar metrics, such as the mean PSF width, or the standard deviation about that mean, as a function of elevation.
Similarly, the sky transparency could be handled by looking at the standard deviation compared to a 2-d map of a photometric night.
Reducing a scalar field to a scalar metric creates a more generalizable framework to communicate data, however, it comes a the expense of removing information.

The most concerning issue with representing a field as a single metric is that it can hide underlying systematics, such as having only one side of the field having an effect, which is not noticed when looking only at a single number representing the entire field.
For this reason, and for the more general reason of needing the ability to dig into the data when a metric is not within the expected range, it is required to have the ability to view and reproduce the data that went into calculating the analysis_tools metric.
`FAFF-REQ-0059`_ has been created to capture the functionality of writing to disk both the calculated metric, and the object that was used to determine it.
This capability is now realized by the refactored analysis_tools design.

When diagnosing the data, the plots and investigations can be time consuming to code and display.
Because in all FAFF related use-cases we are dealing with aggregated data, it would be useful to generate a generic application, most likely in Bokeh, that can present both sky and focal plane aggregated data as a function of a 3rd axis of interest.
This should be carried out with the DM DRP team which also need the same functionality and should therefore use the same toolset.
Naturally, people should be able to fork and customize the app for specific implementations if required, although we expect that the general set of functionalities will be sufficient to support the majority of use-cases.

Functionality of the tool could include:

- Ability to flip through a 2-d data cube as a movie
- Click on a given amp and have a plot of the value versus time, with the expectation value of the metric over plotted etc.
- Ability to show sky maps as a function of time, and adjust the binning on-the-fly
- Capable of mining the appropriate data given the specific analysis_tools metric (including timestamp etc)

Lastly, it is recognized that the DM DRP team also needs to interact with non-aggregated data, this is outside the scope of FAFF, however, adopting a common toolset, or one that is based off the tooling being discussed here is recommended.



.. _Deliverable 5:

Deliverable 5: Computing Resources and Infrastructure
-----------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  5. (`SITCOM-174`_) Using the responses to questions 1-4, propose a management & maintenance structure for the Camera Diagnostic & Commissioning Clusters.

     This includes identifying what processes require specific hardware and/or infrastructure, identifying the more generalized analyses that may benefit from a common infrastructure, and evaluating possible solutions that can ease duplication of effort.

As outlined in the first FAFF report, the primary Chile-based options for `significant computing power <https://sitcomtn-025.lsst.io/#available-computing-power>`_ for commissioning are the Camera Diagnostic Cluster and Antu (often referred to as the Commissioning Cluster).
The summit cluster (Yagan) is also available for use, but is primarily allocated for the control system applications (e.g., LOVE, Sasquatch).


Camera Diagnostic Cluster
^^^^^^^^^^^^^^^^^^^^^^^^^

The Camera Diagnostic Cluster (CDC) is smaller in size than Antu but it has access to the pixel data a few seconds before any other compute resource.
The Camera Diagnostic Cluster is located at the summit, meaning that even in the event of a network failure to the base or USDF, it can continue to function and support both the hardware and observers.
For these reasons, we recommend that the Diagnostic Cluster be used to run the CVT and perform basic calculations to support camera health.
These values will be sent to Sasquatch and recorded in the EFD.
This allows tools such as LOVE and Bokeh Apps to be used for display when required.
With the exception of displays developed and used by the CCS team to support camera operations, we recommend that the Camera Diagnostic Cluster not be used to generate, publish, or visualize plots.
Where possible, this should be accomplished using the common toolsets (e.g., Bokeh).

The Camera Diagnostic Cluster will use a simplified set of tools to perform rudimentary on-the-fly calculations, for example, means and standard deviations of overscan regions.
These analyses will be developed and managed by the camera team.
Using the DM tool set, although useful, would add significant complexity, specifically in regards to maintenance and updates, that would go largely unused if the desire was only to replace the values being calculated now during EO testing.
Instead, those more sophisticated types of calculations will be run using the DM tool set as part of the Rapid Analysis Pipeline.

Antu at the Base (Current Baseline)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The original project plan has Antu residing at the base in La Serena, acting as a general compute facility to support commissioning and summit personnel.
Rapid Analysis would be run on Antu, where there is significantly more computing power and storage than the Camera Diagnostic Cluster.
This has several implications for what happens in the event of a communications outage between summit and base, as discussed in `Deliverable 2`_.
Another way to frame the issue is to consider what is critical to be computed in the event of a connection loss to the Base Facility.
Unfortunately, the definition of what needs to be calculated on the summit to support operations is closely tied to the concept of "Degraded mode," which is currently not sufficiently defined to draw a single conclusion.
Therefore, we consider here three separate states of functionality for the observatory in the event of an outage:

1. The observatory is able to safely continue standard survey operations with minimal functionality to evaluate science data quality in real time.
   Image display is still occurring because the CVT is hosted on the summit-based diagnostic cluster and observers can visually inspect raw images and images with minimal instrument signature removal.
   Low-level calculations and analysis will go into the camera database and the EFD.
2. As above, with the addition of the Rapid Analysis framework to support operations, scheduler input, QA analyses etc.
3. Full operations, including all processing that is planned to be performed at the USDF, such as Alert Processing, with transfer of diagnostic information back to the summit.

State 1:
   The observatory is able to safely continue standard survey operations with minimal functionality to evaluate science data quality in real time.
   Image display is still occurring because the CVT is hosted on the summit-based diagnostic cluster and observers can visually inspect raw images and images with minimal instrument signature removal.
   Low-level calculations and analysis will go into the camera database and the EFD.
State 2:
   As above, with the addition of the Rapid Analysis framework to support operations, scheduler input, QA analyses etc.
State 3:
   Full operations, including all processing that is planned to be performed at the USDF, such as Alert Processing, with transfer of diagnostic information back to the summit.

Maintaining State 3 in the event of a network outage means moving all Alert Processing infrastructure to the summit.
This is not practical for many reasons, nor is it a requirement, and is therefore not discussed further.

In the event of a network failure between summit and base, the observatory would at most be able to achieve State 1.
Because no Rapid Analysis support will be available from the base, any (non-AOS) image-based calculations will not be performed and therefore it is possible that certain engineering tests will not be able to be performed, and (potentially) certain inputs to the scheduler may not arrive.

If we consider that the camera diagnostic cluster could perform some of the tasks considered in State 2, for example, a subset of Rapid Analysis is required (which we refer to as rapid-analysis-critical) to remain functional in the event of an outage, this requires a very significant increase in functionality.

- DM tooling must be installed and maintained on the diagnostic cluster
- Rapid-analysis-critical must be developed and deployed, with the ability to only focus on a subset of detectors, and/or metrics
- The database containing the output must be hosted on the summit, then replicated outwards

Note that the full output of Rapid Analysis cannot be computed due to the limited compute power.

This committee suggests that if Antu does need to stay at the base, then a step-wise approach where the infrastructure for scenario 1 gets implemented prior to significant effort being put into scenario 2, if deemed appropriate.
The preferred solution is to move the Antu servers to the summit.

.. _antu_at_summit:

Antu at the Summit (Proposed Change)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Another possibility which has been considered by this group is to relocate the Antu servers to the summit, even if it means reducing the cluster size in Chile and increasing the capability at the USDF.
This scenario reduces the scope of the commissioning cluster, essentially relocating the functionality of a general compute facility to the USDF, and having the cluster be a more direct support to on-the-fly observations and reductions.
In doing so, this allows States 1 and 2 above to be supported when a network outage to the summit occurs.
Furthermore, it simplifies the number of systems that require support which significantly reduces the workload of the IT group.

The first hurdle of moving Antu to the summit is the capacity to store, power, and cool the servers.
The Chilean IT manager, Christian Silva, informed us that 2500 cores can be run on Cerro Pachón if needed.
However, the support is based around 22 nodes or ~1400 cores, which is Yagan (being upgraded to 640 cores) and Antu (784 cores).
Therefore, capacity is not an issue.
However, we must also consider what computing resources are required to support the two main use-cases for Antu:

1. Running Rapid Analysis and the necessary display tools
2. Being able to run full-focal plane wavefront sensing by pistoning the entire camera in and out of focus

FAFF has shown that item 1 is feasible, which was presented in the `Potential Paths for Implementation`_ subsection of `Deliverable 2: Rapid Analysis Calculated Metrics`_, albeit with a limited number of detectors.
The full focal plane sensing use-case suffers the same limitations of the Rapid Analysis framework, and has an increased computational load.
Currently, the full analysis takes approximately 3 minutes using 2-cores per chip on Antu, and is independent of location.
However, moving the Antu servers to the summit enables this processing to occur in the event of an outage to the base.
Speeding up this process, if required, would necessitate processing the data at the USDF, which is planning real-time support for commissioning (see `RTN-021 <https://rtn-021.lsst.io>`_).
Although this does not explicitly include donut analysis, the cluster is fully capable of doing so and would not be running other real-time analysis at that time.
A trigger to process the AOS data would be required, how this would get accomplished is under investigation.
Discussions are currently ongoing with Richard Dubois to better define the needed support and required timeline(s).

Therefore, FAFF ultimately recommends moving the Antu servers to the summit; the technical details are currently being captured in `ITTN-061 <https://ittn-061.lsst.io>`_.
This will add functionality in the case of an outage and decreases the workload of cluster management and maintenance by co-locating the hardware and removing one set of services.
If the compute load is insufficient to perform all Rapid Analysis tasks, then we can either augment the number of machines, or reduce the number of detectors that are processed in the pipeline.
In discussions with both the AOS and Science Verification teams, using ~50% of the detector has not been met with any resistance.
If full-focal plane wavefront sensing requires more compute, we recommend moving that processing to the USDF and developing an automatic trigger mechanism.
In the case where the link to USDF is lost, it will be required to accept the additional overhead associated with performing the calculation on fewer machines (the Antu servers) [#]_, which is the originally baselined plan.


.. [#] A single full focal plane analysis currently takes ~3 min with 2 cores per chip. Note that Rapid Analysis does not need to be run on these images, thus saving compute time, but it is important to make sure the processes are setup such that they do not compete.

.. RA triggers same as PP.
.. Working on getting data processed via SFP.
.. AOS trigger could be by via OCPS but could be different.


.. _Deliverable 6:

Deliverable 6: Camera Visualization Tool Expansion Support
-----------------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  6. Develop a plan and scope estimate to expand the Camera Visualization Tool to support the full commissioning effort.

     This includes identifying libraries/packages/dependencies that require improvements (e.g. Seadragon) and fully scoping what is required to implement the tool with DM tooling such as the Butler.
     The scope estimate may propose the use of in-kind contribution(s) to this effort if and where applicable.

We have developed a plan to address the visualization requirements developed as part of FAFFv1 and further refined based on
discussion during FAFFv2. The plans include the following major categories:

1. Requirements that can be implemented with existing/planned camera/contributed labor
2. Requirements which require additional hardware at USDF to support
3. Requirements which will need significant front-end work
4. Requirements which require significant DM expertise/assistance

Significant progress has been made on category 1, including effort contributed by Oxford,UK under UKD-UKD-S7.
We have also made progress on item 4, in particular targeting an early proof-of-concept by adding the ability
to display DM generated FITS files including some level of instrument signature removal (ISR)
using the RubinTV generated files from AuxTel. We are developing plans to generalize this work to ComCam and the main camera,
with the intention of using ISR files generated on the commissioning cluster (Antu -- see above).

These plans are being rolled out as incremental improvements to the camera image visualization tool which is
already being used in Chile with AuxTel and ComCam, and at SLAC for the full camera and TS8.

This work is being further tracked under: https://jira.lsstcorp.org/browse/SITCOM-190


.. _Deliverable 7:

Deliverable 7: Catcher Development
----------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  7. Work with project software teams to and implement an initial version of the Catcher CSC and supporting functionality.

     An initial description of required functionality was delivered in the first FAFF charge.
     This deliverable is to implement (at least) two use-cases; one which uses image data and the other which does not.
     Subsequently, suggest a developer and/or in-kind contributor continue development.


.. warning::

   This section is not yet completed and only reports the current status.


The requirements for Catcher were spelled out in the original FAFF report and will not be repeated here, however, it is essentially a service that monitors the control system for specific events and or situations, launches a detailed analysis when those events occur, then produce artifacts and/or alarms when required.
The Catcher is a name that has been assigned to the group of required functionalities and is not necessarily the suggested name for the required tool.
An example of a functionality requiring the use of the Catcher would be if excessive jitter is seen in the telescope encoders that are indicative of an external driving force (e.g. vibration) during a slew.
If one was only interested in image quality, then this analysis could be calculated when an image is taken via the Rapid Analysis framework.
There are many effects need to be acted on that are independent images, and therefore utilize the Catcher.
Lastely, it should be noted that the Catcher is not required to act on results generated by Rapid Analysis, as this would be accomplished using the `analysis_tools` package.

As part of the FAFFv2 effort, other architectures besides a CSC have been explored, specifically using Flux scripts and the InfluxDB architecture, which is designed to do perform analogous use-cases.
The Catcher high-level design work is being documented in `a technote <tstn-034.lsst.io>`_.
The addition of new tools is not being taken lightly, but was originally thought to ease the net complexity of development, usage and maintenance.
At this time, it appears that the fundamental issue with these tools is getting reporting from those analysis back into the control system architecture.
An example of such an interaction is the requirement of being able to report issues to observers via LOVE.
For this reason, it is currently envisioned that the Catcher will have to utilize the CSC architecture, but this is still being explored.

While the design requirements for the Catcher are based upon the numerous FAFF use-cases, the initial design prototype is based upon the execution of two representative scenarios that broadly summarize the main functionalities.
The fundamental difference between the use-cases is the involvement of on-the-fly image processing and interaction with the OCPS.

Example Catcher Non-image Use-case
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This use-case is designed to operate entirely independent of any image taking.

**Trigger:** Telemetry (wind speed) passes threshold. Evaluated on a user-specified time interval (~1 minute).

**Execution (job):** Gathers last ~30 minutes of wind data, fits and extrapolates into the future.
If the estimated wind in ~10 minutes exceeds a user-specified threshold, then an alert is raised to the observer.
The analysis must be persisted, a plot plot showing the extrapolation must be presented to the observer.

**Alert:** User gets notification of probably windshake, with link to webpage

**Implementation for Prototype:** This section has not yet been completed.

Example Catcher Image-based Use-case
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This use-case forces interactions with image telemetry and analysis.
It is anticipated this situation will primarily apply when specialized reductions and/or analyses are required that are not available as part of SFP.
Presumably the calculations will be CPU intensive or they would be done for every exposure.

**Trigger:** An endReadout event from a camera (e.g. LATISS)

**Execution (job):** Gathers data from the EFD, and calculates a metric (e.g. RMS of telescope encoders and the wind speed).
If the metric reports back as True, then a command to the OCPS is sent to start a detailed analysis and persist the result.
From that analysis, if a threshold is surpassed, an alert should be generated for the observer.
Optional: Assembly of an object (artifact) that can be read, processed, and displayed in by a Bokeh app.

**Alert:** If above threshold, user gets notification with link to artifact.
If below threshold, artifact is archived, but no alert is issued.

**Implementation for Prototype:** This section has not yet been completed.


.. _Deliverable 8:

Deliverable 8: Training
-----------------------
.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  8. Design user-level training bootcamps and materials, aimed at the level of an in-kind contributor.

     These bootcamps will be used as the initial training materials.
     It is expected that In-kind contributors and/or other delegates can augment the content, provide improvements, and eventually take over some of the training.

Because much of the required values when dealing with images are calculated by the Rapid Analysis framework, which utilizes pipe tasks, observers nor in-kind contributors can be expected to deliver code.
The most obvious training regarding dealing with Rapid Analysis data is the querying of the database.
However, we expect the implementation is built around the EFD Client or analogous using SQL-like syntax, then no formal training is required.

In similar vein is the usage of the CVT.
This is not sufficiently complex to require special bootcamps.
The CCS team will deliver a user-guide with examples to demonstrate and explain the functionality.

Where special training is required is with regards to use of the Catcher, and the development of custom on-the-fly jobs, generation of artifacts, and alerts to the user.
Because the development of the Catcher framework is in its infancy, a formal training package cannot yet be developed.
However, upon completion, or at least the implementation of an alpha version, a bootcamp, or series of bootcamps, will be necessary that explains the following items:

   - How to create a trigger for a Catcher job based on the evaluation of a boolean condition (e.g. measured value exceeds a threshold)
   - The multiple scenarios in which an analysis job can be written and executed
   - The multiple types of artifacts that can be generated, ranging from a single scalar, to complex data objects, to a png file.
   - How to archive the artifact
   - How to display an artifact, including how to deploy a Bokeh App that utilizes the aforementioned complex data object
   - How to alert a user, specifically an operator, that an artifact is available for viewing (with a level of urgency attached)


Useful trainings, but arguably out of FAFF scope also include trainings in preparing for an observation, writing SAL scripts, and operating the telescope via LOVE.
Also out of scope, but useful to commissioning personnel, are the writing of modules and/or pieces of code that can be added to the Science Pipelines to support FAFF needs, for example, using the analysis_tools package to create science performance metrics and diagnostic plots, or adding additional data quality statistics to SFP.


.. _Deliverable 9:

Deliverable 9: Task Prioritization
----------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  9. A prioritized list of tasks to build-out the new functionalities with recommended end-dates.

     Where possible, these dates shall correspond to integration milestones.


Because much of the work is highly parallelizable, this report groups tasks into prioritization tiers; tasks are not ranked individually within a given tier.
These tasks are focused on developing high-level architectures.
The list does not include all the specific displays and/or toolsthat are required for each individual system (e.g. the AOS GUIs).
However, a non-exhaustive list of these tools are discussed in the `Recommended Tools`_ section.
Lastly, the reader should recognize that there is a lot of work to be accomplished that can only be done by small and specific groups of individuals.
Coordination and management of these tasks will be critical to success of commissioning.


Tier 1:
^^^^^^^

- Complete transition of the Antu servers to the summit.
  This task is required before many of the Tier 2 tasks can make significant process.
  This is because because the Rapid Analysis framework will run on this cluster.

Tier 2:
^^^^^^^
The following are in order of importance, but again are largely parallelizable between various parties.

- Setup and configure the Camera Diagnostic Cluster.
  This includes running and publishing the low level diagnostics, then progressing on the alert infrastructure.
-  Deploy the Rapid Analysis framework at the summit
   Initial efforts should be focused on development of interfaces and not speed.
   Capabilities of each part can be expanded incrementally.
   Early testing can store metric values in the Butler.
- Create a database in Sasquatch for recording the Rapid Analysis Metrics
- Define alert framework for the alarming metrics and for when processed images are available

Once the above are completed, then the following can be performed:

- Record Rapid Analysis Metrics in the Summit Sasquatch instance
- Replicate Rapid Analysis Metrics from the Summit Sasquatch instance to the USDF Sasquatch instance
- Start issuing alerts that metrics and processed images are available
- Create visualizations w/ Chronograf, etc.

Tier 3:
^^^^^^^

- Development of the Catcher
- Performing daily `12-24 hours`_ SFP at USDF.
  This capability is not required to support realtime decisions during nighttime operations.
  There is requirement to wait until the morning to begin reductions and in fact processing the results as the data streams in is preferred.
  Note that early runs can put data into the butler and can then be expanded to a database.
- CVT augmented to read processed images from Rapid Analysis, then expanded to support any full frame image persisted in the Butler
- Create templates for development of Catcher, Bokeh, and possibly LOVE displays by SIT-Com personnel and/or project software developers.
- Develop training examples.
  Ideally, this will be performed in conjunction with the development of templates.

Again, developing a common toolset between the commissioning team and the DRP, or one that is based off the tooling being discussed here, is strongly recommended.
This is not explicitly listed as a priority as it should be a continually ongoing activity.

Recommended Tools
^^^^^^^^^^^^^^^^^

Once the frameworks defined above and prior to entering commissioning, a series of additional tools need to be constructed to facilitate commissioning.
The following is a non-exhaustive list of general tools that will be required and are not currently owned by any person and/or group.
It does not include subsystem specific displays such as what will be required for commissioning the Active Optics System.

#. An on-the-fly telescope offset calculation and implementation tool.
#. A tool to display scalar fields, as discussed in `Deliverable 4`_.
#. A display showing the calculated metrics for each image, with indicators when values are out of range.
   The contents should be linked to down-range diagnostic tools/displays that are accessed upon "clicking."
#. Strip charts showing data quality metrics versus observing conditions.
#. Image summary "pages" that display basic parameters, such as the PSF fundamental properties, filter used, observatory setup etc.
   Such as is done for Rubin TV.
#. Logging tool that relates a obs-id (or other unique identifier) to all of the different areas having artifacts.
   Similarly, the logging tool should also allow items that are not directly related to an image ID.
#. Need a tabular view that relates images to all of the metrics and available plots/data/artifacts, analogous to what is `used for HSC <https://confluence.lsstcorp.org/display/LSSTCOM/Lessons+learned+from+HSC+commissioning+and+operation+in+terms+of+On-the-fly+Analysis+Use-Case>`_.
#. Generic webpage containing links to commonly used, but (normally) external tools.
   We started a `website <https://obs-ops.lsst.io>`_ to host such data, it is meant to be observer focused and is currently being better populated, however, a more global effort is required.


Multiple databases that need merging:

1. Scheduler database
2. Exposure Log database (camera)  - drives camera visualization


.. _Derived Requirements:

Generated Requirements
======================

Based upon the above use-cases, numerous requirements on to-be-designed and implemented systems have been derived.
This section captures these and roughly organizes them by application.

The requirements below are in addition to what was presented in the first `FAFF report`_.

Processing
----------

FAFF-REQ-0051
^^^^^^^^^^^^^
**Specification:** The Rapid Analysis of images shall maintain the same cadence as the telescope visits.

**Rationale:** The data reduction and analysis must not fall behind the data being taken.
Frames should not be skipped in order to catch up.

FAFF-REQ-0052
^^^^^^^^^^^^^
**Specification:** A full SFP shall be run on images within 24 hours of observation.

**Rationale:** Ideally this would be done in less than 12 hours, so people could look at it before the next night's observation, although this is a stretch goal.
This data uses the most recent (best) science pipelines and produces the highest quality data products that are used for science verification tasks.


FAFF-REQ-0053
^^^^^^^^^^^^^
**Specification:** Rapid Analysis shall produce data products that are not required to continue data taking during commissioning nor operations.

**Rationale:** The telescope need not stop observing if the Rapid Analysis fails.
However, it is expected that functionality may be reduced and/or the planned observations/activities may change.


FAFF-REQ-0057
^^^^^^^^^^^^^
**Specification:** Rapid Analysis data processing (and storage) shall only be run once.

**Rationale:** This is a one-off on-the-fly analysis.
Data products, even if incorrect, will remain as such.
This is intentional to keep a record of what was available to the user (and/or scheduler) at a later time.
Because Rapid Analysis is not re-run, no versioning or relationships to other calculated results in the future need to be supported.


FAFF-REQ-0058
^^^^^^^^^^^^^
**Specification:** Observers shall be able to run instances of SFP manually to support commissioning.

**Rationale:** If Rapid Analysis fails, then users will need the capability to re-run the analyses.
This is expected to be done either at the USDF or on the commissioning cluster.
The results are not to go into the Rapid Analysis (or any other shared) database.
It is expected that this is essentially a single line of code, but will require training.


FAFF-REQ-0059
^^^^^^^^^^^^^
**Specification:**  Observers shall be able to reproduce analysis_tools metrics and the data that went into them.

**Rationale:** The metrics are scalars and therefore do not include all required information to diagnose a problem.
One way to satisfy this requirement is to ensure that the "analysis_tools metric modules" are importable and the objects use to determine them are either stored, or at a minimum are easily reproduced.

Data Access
-----------

FAFF-REQ-0054
^^^^^^^^^^^^^
**Specification:** All processed data and artifacts shall be referenced from a single source, as viewed from the user.

**Rationale:** Users will need to access telemetry data, Rapid Analysis data, and all generated artifacts in the same manner.
They need not be pre-occupied with where the data exists and why.
This requirement does not specify everything must be stored in a single database, although it may be a solution.
It is also acceptable that a query returns a link to a file in the LFA.


FAFF-REQ-0055
^^^^^^^^^^^^^
**Specification:** The Rapid Analysis processed data and artifacts must be accessible from the major data processing facilities (e.g. Summit, Base, USDF).

**Rationale:** This will probably require replication of the data, analogous to Sasquatch for replicating the EFD data.


FAFF-REQ-0056
^^^^^^^^^^^^^
**Specification:** The Scheduler must be able to access the Rapid Analysis database.

**Rationale:** If the database is implemented in Sasquatch a mechanism to access the data already exists.
The Scheduler database is currently independent and needs to be merged.


Display Tooling Requirements
----------------------------

Most display tooling requirements are found in the first `FAFF report`_.

FAFF-REQ-0060
^^^^^^^^^^^^^

**Specification:** Display a histogram based on selected region.

**Rationale:** This is a functionality widely used by the camera team.

..
   FAFF-REQ-XXXX
   ^^^^^^^^^^^^^
   **Specification:**

   **Rationale:**



.. _Other Findings and Identified Issues:

Other Findings and Identified Issues
====================================

During the existence of this working group, numerous items were identified as problematic and needing to be addressed but either were not well fit to a charge question or fell out of the scope of the charge.
This section contains information regarding numerous issues which were identified and require attention.

- Lack of definition regarding degraded mode(s)
