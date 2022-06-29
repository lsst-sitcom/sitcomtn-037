:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

.. _SITCOM-173: https://jira.lsstcorp.org/browse/SITCOM-173
.. _SITCOM-174: https://jira.lsstcorp.org/browse/SITCOM-174
.. _SITCOM-180: https://jira.lsstcorp.org/browse/SITCOM-180
.. _charge: https://sitcomtn-030.lsst.io/
.. _FAFF report: https://sitcomtn-025.lsst.io/

Abstract
========

The First-Look Analysis and Feedback Functionality Breakout Group `charge`_ 2 details questions regarding the displays and functionalities required to perform continuous nightly commissioning activities.
The focus is on deriving the required functionalities to perform inspection and analyses within approximately 24-hours of taking the data.
This document details the answers to the charge questions and other findings.

Executive Summary
=================

TBR


Introduction
============

This report addresses the second charge to the First-Look Analysis and Feedback Functionality (FAFF) Breakout Group.
The charge was developed based on the findings during the Missing Functionality Workshop that took place February 2nd and 3rd, 2022.
The charge addresses three specific Jira tickets developed from that meeting: `SITCOM-174`_, `SITCOM-173`_, and `SITCOM-180`_.
These are referenced throughout the report.

Much of this report builds off the findings and recommendations of the first `FAFF report`_.
It is expected that the readers are already familiar with the findings and recommendations.

This report is structured into sections, where the first addresses each of the charge questions individually.
The charge question has been copied into each section for ease-of-readability.
After addressing each charge question is a section on findings that are pertinent to the commissioning team but fall outside the scope of the charge.
It is recommended that these issues get addressed by either a follow-on charge or another mechanism.


Responses to Charge Questions and Deliverables
==============================================

The use-cases referenced throughout the document are all found on the :ref:`pg-D1-Use-cases` page.

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
One major difference is that nearly all use cases made the assumption that there would be a process which created data products based on observatory telemetry and camera images that would be used as the basis for displaying information.
Rather than each use-case stating the needed data products and explaining their generation mechanism, we created a single use-case that documents the needed values and timescales for their creation.
We call this processing step `Rapid Analysis <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`, and the data products are listed explicitly, as requested in charge question 2.

The remaining use-cases for FAFF2 can be found at the bottom of the page `on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Use-Cases>`_ and are referenced throughout the remainder of this report.


.. _Deliverable 2:

Deliverable 2: Rapid Analysis Calculated Metrics
------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   2. (`SITCOM-180`_, `SITCOM-173`_) Define which metrics, analyses and artifacts must be calculated and on what timescale they must be evaluated and reported to support commissioning/operations.

      This is to evaluate if a "rapid processing" of data is required, what specific calculations are required.
      This list should include the relevant camera specific calculations (which are currently performed by the EO testing data reduction).
      This is expected to inform the answer to the next charge task.


Numerous calculations are required to evaluate camera and system health and performance on rapid timescales.
The data products discussed in this section are limited to scalars and/or arrays and does *not* include the generation of plots and/or figures.
The high majority of the data needed on rapid timescales is consistent with a pared down version of the data products produced by the single-frame-processing (SFP) framework.
A small number of additional values are also required, but can be quickly derived from the SFP results.
The calculated values from Rapid Analysis are not to produce data products that are critical to commissioning (`FAFF-REQ-XXX3`_), however, it is expected that observatory functionality is reduced if an outage were to occur.
This implies that the Rapid Analysis is not required to run at the summit, although if would be preferable to do so.
The output from the Rapid Analysis will need to go into a database.
Details of this are discussed in

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
Further details regarding use of the commissioning cluster are discussed in `Deliverable 5: Computing Resources (Clusters)`_.

The SFP pipeline is to be run on Antu (the commissioning cluster), where more compute is available and the hardware consists of generic and more easily managed servers.
There are values in the SFP pipeline that are more pertinent to have on shorter timescales, such as the PSF shape.
These values have been identified in the `Rapid Analysis Use-case <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_ and if it is possible to output them prior to others it would help increase operational efficiency.

~60 seconds
^^^^^^^^^^^
This timescale is useful when examining trending or slowly varying effects, particularly for metrics like image quality or transparency.
It is a timescale where people are closely watching, but not necessarily immediately reacting.
The addition of this category was to provide flexibility in implementation as it may be such that the prioritization of metrics can be performed which may provide a useful free parameter during the implementation phase.
However, it is imperative that the rapid analysis framework be able to keep up with the rate of images being acquired; where that rate is governed by the survey strategy visit duration (`FAFF-REQ-XXX1`_).
In the case of taking two 15 second snaps, it is expected that the analysis would be done on the combined images.

Again, the data products for the 60 second timescales are described in the Outputs section of the `Rapid Analysis Use-case <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_.


12-24 hours
^^^^^^^^^^^
This timescale is important for more general commissioning activities and performance assessment that could impact observations taken in the next or subsequent nights.
Over this timescale, a full DRP single frame processing pipeline needs to be run (`FAFF-REQ-XXX2`_).
This must include the additional values that are calculated in the Rapid Analysis Framework, which will need to be added to the SFP pipeline.
Re-calculation of these values enables a more detailed and higher-confidence data quality evaluation to be performed, including correlation with telemetry, environmental conditions, and previous conditions and/or observations.
It also allows the teams to begin determining which subsets of data should be used to construct coadds/templates, begin SV analyses, and ultimately maximize the number of human brain cycles looking at the data.
It is fully expected that this dataset will be superseded by a subsequent DRP campaign to enforce that all the data is processed in a homogeneous way with best performing configuration of the science pipelines.

It is not required that the full SVP processing be done in Chile, in fact, it is *preferable* to perform this processing at the USDF as many of the science verification tasks are planned to be performed there as well.
It also ensures that a minimum number of users are connecting to Chile to perform their analysis.
This is especially important if connections would be required to the summit instance.

Potential Paths for Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The rapid analysis framework relies heavily on single frame processing, and therefore is very compatible with both the DRP and the Alert Production Pipelines.
However, because of the speed requirements, which will necessitate the pre-loading of expected image properties into memory (e.g. catalogues), it is expected that the path of least resistance would be to work with the APP team in the development of rapid analysis.

FAFF has worked with Rubin project members to create a preliminary analysis of the compute required to run Rapid Analysis and found that with ~250 cores (1 per detector and a handful for overhead), combined with some attention paid to code performance enhancements, we expect that in terms of processing, keeping up with a 30s image cadence is very feasible.
The University of Washington group is now investigating the SFP performance enhancements (FIXME - update when report conclusion is known).
Arguably, the more challenging aspect is the reading and writing of files to and from the disk.
FIXME: UPDATE WHEN ACTION ITEM COMPLETED.


Another important point is that Rapid Analysis only needs to run once per frame. Even upon a failure to produce one of the parameters, or the publishing of an incorrect result, the system will not be rerun and therefore the database containing the results does not need to support versioning or relationships to previous results.


.. _Deliverable 3:

Deliverable 3: Interacting with Rapid Analysis Data and Metrics
---------------------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   1. (`SITCOM-174`_, `SITCOM-173`_) Define how users will interact with each aspect of the previously listed metrics, analyses and artifacts; classify them indicating where can could calculated.

      This includes tasks defined for the catcher, OCPS jobs, AuxTel/ComCam/LSSTCam processing, and the rendez-vous of data from multiple sources (DIMM, all-sky etc).


Simple scalar metrics (e.g. DIMM measured seeing) are easily captured in things like Chronograf, and are not addressed here.
They can be considered a subset of the scalar fields case below.

This section considers the case of scalar fields --> Displaying metrics as a fxn of position/amp/detector etc.


This is:

- camera visualization health tool(s).
- Scheduler Troubleshooting
- Extended functionality of the CVT (but better captured in the section, `Deliverable 6`_)
- Bokeh Apps
- Webpages (needs expansion on how this would be used, Noteburst+Times Square is an option)
- Trending plots (see also `Deliverable 4`_ for discussion of scalar fields as a function of a 3rd axis)

Useful to group into aggregated (binned) and non-aggregated (unbinned) metrics.

- Binned: aggregated values that are pre-computed on a certain spatial scale (e.g. an amp/detector/raft) data where the scaling can be changed and could be modified to varying scales
- Unbinned: Value per source (e.g. photometry measurement at each previous visit).
 Depending on the case, a slider could be present to adjust the scaling on-the-fly

After significant discussion, it was determined that operations on the mountain and within the first ~24 hours of taking data, it is sufficient to deal with ONLY aggregated data.
However, multiple forms of aggregation need to be supported (per amp, per detector, per raft, per healpix, sq degree etc.)




Databases
^^^^^^^^^

Data from the observatory will come from numerous sources and efforts should be made to minimize the number of individual databases; both for maintenance and ease-of-use reasons.
Whereas much of the data coming off the summit is time based, and therefore goes into a time-based database (the EFD), other aspects of the system are image based, such as what will be produced by Rapid Analysis and the parts of the camera system.

- Rapid Analysis data needs to go into a database. Database implementation is beyond FAFF scope, but regardless of implementation users need a framework/method that manages the point(s) of access, analogous to the EfdClient (`FAFF_REQ_XXX5`_)
- The database must be available at all major data facilities (`FAFF-REQ-XXX5`_), analogous to what is done for the EFD.
- Summit tooling, including the Scheduler, must have immediate access to the database (`FAFF-REQ-XXX6`_).

To aid in the implementation/amalgamation of databases, each use-case has a section with example queries of how the information might be accessed (FIXME: TBR when Robert has completed his action item).



.. _Deliverable 4:

Deliverable 4: Required Non-Scalar Metrics
------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  4. (`SITCOM-180`_) Provide a list of required non-scalar metrics are required and cannot be computed with faro.
     Suggest a mechanism (work flow) to perform the measurement, document the finding, evaluate any trend (if applicable), then present it to the stakeholders.


.. related to https://confluence.lsstcorp.org/display/LSSTCOM/Displaying+scalar+fields+as+a+function+of+other+parameters

This charge question covers the issue of calculating and displaying the trending of scalar fields.
Scalar fields are single value metrics, but calculated per spatial element, as described in `Deliverable 4`_.
This charge question deals with adding a third dimension to the scalar field, then calculating and displaying this data to the user.
For example, this could be displaying the PSF width for each detector as a function of elevation, or sky transparency as a function of time.
As discussed above, both of these examples deal with aggregated (binned) data.
Faro computes single valued (scalar) metrics and compares against an expected value or range (e.g. a sigma or mean), however, it can present this data in aggregated ways (such as per detector, per raft etc.).

After analyzing the use-cases, including hypotheticals not detailed in the report, it was decided that there is not a use-case where we are unable represent a scalar field with respect to a third axis (e.g. time, elevation etc) as a single valued metric (e.g. a mean, or standard deviation), so long as the desired aggregation is supported.
Taking the examples discussed above, one would reduce the scalar field to a number of scalar metrics, such as the mean PSF width, or the standard deviation about that mean, as a function of elevation.
Similarly, the sky transparency could be handled by looking at the standard deviation compared to a 2-d map of a photometric night.
Reducing a scalar field to a scalar metric creates a more generalizable framework to communicate data, however, it comes a the expense of removing information.

The most concerning issue with representing a field as a single metric is that it can hide underlying systematics, such as having only one side of the field having an effect, which is not noticed when looking only at a single number representing the entire field.
For this reason, and for the more general reason of needing the ability to dig into the data when a metric is not within the expected range, it is required to have the ability to view and reproduce the data that went into calculating the faro metric.
`FAFF-REQ-XYZZ`_ has been created to capture the functionality of writing to disk both the calculated metric, and the object that was used to determine it.

When diagnosing the data, the plots and investigations can be time consuming to code and display.
Because in all FAFF related use-cases we are dealing with aggregated data, it would be useful to generate a generic application, most likely in Bokeh, that can present both sky and focal plane aggregated data as a function of a 3rd axis of interest.
This should be carried out with the DM DRP team which also need the same functionality and should therefore use the same toolset.
Naturally, people should be able to fork and customize the app for specific implementations if required, although we expect that the general set of functionalities will be sufficient to support the majority of use-cases.

Functionality of the tool could include:

- Ability to flip through a 2-d data cube as a movie
- Click on a given amp and have a plot of the value versus time, with the expectation value of the metric over plotted etc.
- Ability to show sky maps as a function of time, and adjust the binning on-the-fly
- Capable of mining the appropriate data given the specific faro metric (including timestamp etc)

Lastly, it is recognized that the DM DRP team also needs to interact with non-aggregated data, this is outside the scope of FAFF, however, adopting a common toolset, or one that is based off the tooling being discussed here is recommended.



.. _Deliverable 5:

Deliverable 5: Computing Resources and Infrastructure
-----------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  5. (`SITCOM-174`_) Using the responses to questions 1-4, propose a management & maintenance structure for the Camera Diagnostic & Commissioning Clusters.

     This includes identifying what processes require specific hardware and/or infrastructure, identifying the more generalized analyses that may benefit from a common infrastructure, and evaluating possible solutions that can ease duplication of effort.

As outlined in the first FAFF report, the primary Chile-based options for `significant computing power <https://sitcomtn-025.lsst.io/#available-computing-power>`_ for commissioning are the Camera Diagnostic Cluster and Antu (often referred to as the Commissioning Cluster).
The summit cluster (Yagan) is also available for use, but is primarily alloted for the control systems, LOVE, the EFD etc.


Camera Diagnostic Cluster
^^^^^^^^^^^^^^^^^^^^^^^^^

The Camera Diagnostic Cluster (CDC) is smaller in size than Antu but it has one unique capability in that it has access to the pixel data a few seconds before anything else.
One other advantage is it's location.
Being on the summit means that even in the event of a network failure to the base or USDF, it can continue to function and support both the hardware and observers.
For these reasons, we recommend that the Diagnostic Cluster be used to run the CVT and perform basic calculations to support camera health.
Summary values will be published to DDS, and therefore the values will be archived in the EFD.
This allows tools such as LOVE and Bokeh Apps to be used for summary displays when required.

Camera diagnostic cluster is to use a simplified set of tools to perform rapid yet rudimentary on-the-fly calculations, all managed by the camera team.
Examples include calculations like means and standard deviations of active and overscan regions etc, and will include simple analysis designed to evaluate camera health,
e.g. PSF perhaps calculated on a rotating subset of CCDs.
Using the DM toolset, although useful, would add significant complexity, specifically in regards to maintenance and updates, that would go largely unused if the desire
was only to replace the values being calculated now during EO testing. Although the primary output of the diagnostic cluster analysis will be summary metrics published to the EFD,
it may be useful to additionally create and publish a small set of plots, similar to those currently generated by EO testing. Although ideally we would have common way of publishing such plots
and making them available to operators and camera experts, in the absence of such a mechanism such plots will be stored on the diagnostic cluster and made available via camera web interface
similar to the existing EO summary plots.
Once all such plots are generated using the DM toolset as part of the Rapid Analysis Pipeline the need for generating plots on the diagnostic cluster may diminish or go away.

Antu at the Base
^^^^^^^^^^^^^^^^

The original project plan has Antu residing at the base, acting as a general compute facility to support commissioning and summit personnel.
Rapid analysis is to be run on Antu, where there is significantly more computing power and storage which has several implications, specifically in regards to what happens in the event of an outage.
Another way to frame the issue, is to consider what is critical to be computed in the event of a connection loss to the Base Facility.
Unfortunately, the definition of what needs to be calculated on the summit to support operations is very heavily tied to the concept of "Degraded mode," which is currently not sufficiently defined to draw a single conclusion.
Therefore, we consider here two separate scenarios:

1. Degraded mode means the observatory is able to continue standard survey operations, but at an increased risk of obtaining poor data because of a disconnection from the rapid analysis framework.
2. Degraded mode requires the observatory continue to receive output from the rapid analysis framework to support operations, scheduler input, or potentially QA analyses etc.

In the case of scenario 1, in the event of a network outage, the summit-based diagnostic cluster will run the CVT, and continue to perform the low-level calculations that will go into the camera's database and the EFD.
Observers will see the images being output and be able to visually evaluate performance.
Because no rapid-analysis support will be available from the base, any (non-AOS) image-based calculations will not be performed and therefore it is possible that certain engineering tests will not be able to be performed, and (potentially) certain inputs to the scheduler may not arrive.

In scenario 2, where a subset of rapid analysis is required (which we refer to as rapid-analysis-critical) to remain functional in the event of an outage, this requires a very significant increase in functionality.
- DM tooling must be installed and maintained on the diagnostic cluster
- Rapid-analysis-critical must be developed and deployed, with the ability to only focus on a subset of detectors, and/or metrics
- The database containing the output must be hosted on the summit, then replicated outwards

Note that in the event of scenario 2, certain tasks will still be limited as the computer power is significantly lower which will result in the computations taking longer, and being only a subset.

Regardless of how degraded mode is ultimately defined, this committee recommends that if Antu does need to stay at the base, then a step-wise approach where the infrastructure for scenario 1 gets implemented prior to significant effort being put into scenario 2, if deemed appropriate.


Antu at the Summit
^^^^^^^^^^^^^^^^^^

Another possibility which has been considered by this group is to relocated Antu to the summit, even if it means reducing the cluster size in Chile and increasing the capability at the USDF.
This scenario also reduces the scope of the commissioning cluster, essentially relocating the functionality of a general compute facility to the USDF, and having the cluster be a more direct support to on-the-fly observations and reductions.
For this to be feasible, we must consider what computing resources are required to support the two main use-cases for Antu:

1. Running rapid analysis and the necessary display tools
2. Being able to run full-focal plane wavefront sensing by pistoning the entire camera in and out of focus

FAFF has shown that item 1 is feasible, which was presented in the `Potential Paths for Implementation`_ subsection of `Deliverable 2: Rapid Analysis Calculated Metrics`_.

The full focal plane sensing..... FIXME: Awaiting information from RHL  and the I/O from the RA characterization.


Of course, the projects also needs to have the capacity to store, power, and cool the machines at the summit.
In discussions with Christian Silva, the Chilean IT manager, he informed us that 2500 cores can be run on Cerro Pachón if needed.
FIXME: GET A MORE OFFICIAL STATEMENT from Cristian regarding current load versus capacity.

FIXME: Moving RA to the summit relaxes much of the concern regarding the lack of definition on degraded mode. Also makes cluster management and maintenance easier by co-locating hardware and services.

FIXME: FAFF recommends moving to the summit, even if it means the AOS calculation needs to be run at USDF.

Degraded mode then will still have IQ feedback to observers.

.. _Deliverable 6:

Deliverable 6: Camera Visualization Tool Expansion Support
-----------------------------------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  6. Develop a plan and scope estimate to expand the Camera Visualization Tool to support the full commissioning effort.

     This includes identifying libraries/packages/dependencies that require improvements (e.g. Seadragon) and fully scoping what is required to implement the tool with DM tooling such as the Butler.
     The scope estimate may propose the use of in-kind contribution(s) to this effort if and where applicable.

This is Tony and Gregory to come up with a first crack at this.
Tony already has a document with questions/issues; now discussing with Gregory

.. _Deliverable 7:

Deliverable 7: Catcher Development
----------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  7. Work with project software teams to and implement an initial version of the Catcher CSC and supporting functionality.

     An initial description of required functionality was delivered in the first FAFF charge.
     This deliverable is to implement (at least) two use-cases; one which uses image data and the other which does not.
     Subsequently, suggest a developer and/or in-kind contributor continue development.


Does the catcher have to be able to react to results "published" by rapid analysis?
No. No circular dependencies, faro is the afterburner working with rapid analysis results.

Tiago working on a proposed high-level design for this is in consultation with Angelo.
The proposed implementation is described in `a technote <tstn-034.lsst.io>`_.
A prototype now needs to be developed.

Following use cases are meant to flush out section 3 of https://tstn-034.lsst.io/#service-architecture

Use-case 1 (non-images)
----------
Trigger (flux script): Telemetry (wind speed) passes threshold. Evaluate on a set time interval (1 minute).

Execution (job): Gathers last ~30 minutes of wind data, fits and extrapolates. If estimated wind in ~10 minutes exceeds threshold then alert user. Persist and publish the plot showing extrapolation.

Alert: User gets notification of probably windshake, with link to webpage

Implementation for Prototype
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- trigger implemented in flux
-



Use-case 2 (images)
----------
Based on mount jitter

Trigger: on end-readout event

Execution (job): gathers data from EFD, calculates a metric (RMS), persists the result, determines if it needs to be reported to an observer.
Optional: assembles an object (artifact) that can be read in by a bokeh app (could also be computed in Bokeh)

Alert: Send message (slack ok) to demonstrate ability to send alerts to an arbitrary "service" (e.g. LOVE etc)

Display: Bokeh App

Implementation for Prototype
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- trigger implemented in flux
-


.. _Deliverable 8:

Deliverable 8: Training
-----------------------
.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  8. Design user-level training bootcamps and materials, aimed at the level of an in-kind contributor.

     These bootcamps will be used as the initial training materials.
     It is expected that In-kind contributors and/or other delegates can augment the content, provide improvements, and eventually take over some of the training.

Because much of the required values when dealing with images are calculated by the rapid analysis framework, which utilizes pipe tasks, observers nor in-kind contributors can be expected to deliver code.
The most obvious training regarding dealing with rapid analysis data is the querying of the database.
However, so long as the implementation is built around an efd-client type tool, or even standard SQL queries, then no formal training is required.

In similar vein is the usage of the CVT.
This is not sufficiently complex to require special bootcamps.
The CCS team will deliver a user-guide with examples to demonstrate and explain the functionality.

Where special training is required is with regards to use of the Catcher, and the development of custom on-the-fly jobs, generation of artifacts, and alerts to the user.
Because the development of the Catcher framework is in its infancy, a formal training package cannot yet be developed.
However, upon completion, or at least the implementation of an alpha version, the following training materials will be required:

#. A bootcamp, or series of bootcamps, that explains:

   - How to create a trigger for a Catcher job based on the evalation of a boolean condition (e.g. measured value exceeds a threshold)
   - The multiple scenarios in which an analysis job can be written and executed
   - The multiple types of artifacts that can be generated, ranging from a single scalar, to complex data objects, to a png file.
   - How to archive the artifact
   - How to display an artifact, including how to deploy a Bokeh App that utilizes the aformentioned complex data object
   - How to alert a user, specifically an operator, that an artifact is available for viewing (with a level of urgency attached)


Useful trainings, but arguably out of FAFF scope also include trainings in preparing for an observation, writing SAL scripts, and operating the telescope via LOVE.
Also out of scope, but useful to commissioning personnel, are the writing of modules and/or pieces of code that can be added to rapid analysis and single-frame-measurement.


.. _Deliverable 9:

Deliverable 9: Task Prioritization
----------------------------------

.. note::

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  9. A prioritized list of tasks to build-out the new functionalities with recommended end-dates.

     Where possible, these dates shall correspond to integration milestones.


The following tasks are highly parralelizable, but are listed in series in rough order of importance.

#. Define computing resources strategy (Deliverable 6)
   - Stil evaluating if Antu (the commissionign cluster) can be relocated to the summit
   - Seems probably computationally, but I/O challenges remain to be evaluated.
#. Get basic camera diagnostics running at the summit on the camera diagnostic cluster
#. Get catcher deployed (needed for telescope engineering).
#. Get Rapid Analysis Framework deployed including the supporting database

   - Start with the on-the-fly processing with the goal of developing interfaces.
     Speed enchancements can be left for a later date.
   - Complete a chain starting with a very fundamanetal processing
     This could be just a basic ISR and a single metric (e.g. mean signal per amp) that can be used to help flush out interface(s)
     - metric goes into a "supporting database"
     - Need "alert" or event saying the metric is available
     - Visualize the metric in Chronograf (or some pre-existing tool)

     Can then parralelize the expansion of each of the individual pieces

#. USDF daily SFP "pipeline" can get started
   - data can be pushed through pipelines and just leave the data in the butler
   - Can then be expanded to "publish" the data in the "rapid analysis database"
#. Create templates for development of Catcher, Bokeh, and possibly LOVE displays
#. Develop training examples (actually performed in conjunction with the previous)

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
#. Logging tool that relates a obs-id (or whatever) to all of the different areas having artifacts.  FIXME: How do we handle things that are not related to an image ID
#. Need a tabular view that relates images to all of the metrics and available plots/data/artifacts, analogous to what is `used for HSC <https://confluence.lsstcorp.org/display/LSSTCOM/Lessons+learned+from+HSC+commissioning+and+operation+in+terms+of+On-the-fly+Analysis+Use-Case>`_.
#. Generic webpage containing links to commonly used, but (normally) external tools.
   We started a `website <obs-ops.lsst.io>`_ to host such data, Alysha Shugart and Ioana Sotuela have taken over making it more observer/user friendly and better populated; a more global effort is required.


Multiple databases that need merging:

1. Scheduler database
2. Exposure Log database (camera)  - drives camera visualization


.. _Derived Requiremends:

Generated Requirements
======================

Based upon the above use-cases, numerous requirements on to-be-designed and implemented systems have been derived.
This section captures these and roughly organizes them.

Processing
----------

FAFF-REQ-XXX1
^^^^^^^^^^^^^
**Specification:** The Rapid Processing of images shall maintain the same cadence as the telescope visits.

**Rationale:** The data processing must not fall behind the data being taken.
Frames should not be skipped in order to catch up.

FAFF-REQ-XXX2
^^^^^^^^^^^^^
**Specification:** A full single-frame-processing shall be run on images within 24 hours of observation.

**Rationale:** Ideally this would be done in less than 12 hours, so people could look at it before the next night's observation, although this is a stretch goal.
This data uses the most recent (best) science pipelines and produces the highest quality data products that are used for science verification tasks.


FAFF-REQ-XXX3
^^^^^^^^^^^^^
**Specification:** Rapid analysis shall produce data products that are not critical to operations/commissioning.

**Rationale:** The telescope need not stop observing if the rapid analysis fails, however, it is expected that functionality may be reduced and/or the planned observations/activities may change.


FAFF-REQ-XXX4
^^^^^^^^^^^^^
**Specification:** All processed data and artifacts shall be referenced from a single source, as viewed from the user.

**Rationale:** Users will need to access EFD data, rapid processing data, and all generated artifacts in the same manner.
They need not be pre-occupied with where the data exists and why.
This requirement does not specify everything must be stored in a single database, although it may be a solution.
It is also acceptable that a query returns a link to a file in the LFA.


FAFF-REQ-XXX5
^^^^^^^^^^^^^
**Specification:** The rapid analysis processed data and artifacts must be accessible from the major data processing facilities (e.g. Summit, base, USDF).

**Rationale:** This will probably require replication of the data, analogous to the EFD.


FAFF-REQ-XXX6
^^^^^^^^^^^^^
**Specification:** The Scheduler must be able to access the Rapid Analysis database.

**Rationale:** If the database is the EFD then this is already done.
The Scheduler database is currently independent and needs to be merged.


FAFF-REQ-XXX7
^^^^^^^^^^^^^
**Specification:** Rapid analysis data processing (and storage) shall only be run once.

**Rationale:** This is a one-off on-the-fly analysis.
Data products, even if incorrect, will remain as such.
This is intentional to keep a record of what was available to the user (and/or scheduler) at a later time.
Because rapid analysis is not re-run, no versioning or relationships to other calculated results in the future need to be supported.


FAFF-REQ-XXXX
^^^^^^^^^^^^^

PI: I'm not sure this is a necessary requirement.
Also, if the rapid analysis has something special it calculates, how can it be recalculated?

**Specification:** Observers shall be able to run instances of single-frame-processing manually to support commissioning.

**Rationale:** If rapid analysis fails, then users will need the capability to re-run the analyses.
This is expected to be done either at the USDF or on the commissioning cluster.
It is expected that this is essentially a single line of code, but will require training.


Display Tooling Requirements
----------------------------

FAFF-REQ-XYZZ
^^^^^^^^^^^^^
**Specification:**  Observers shall be able to reproduce faro metrics and the data that went into them.

**Rationale:** The metrics are scalars and therefore do not include all required information to diagnose a problem.
One way to satisfy this requirement is to ensure the "faro metric modules" are importable and the objects use to determine them are either stored, or at a minimum are easily reproduced.




FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:**

**Rationale:**

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:**

**Rationale:**

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:**

**Rationale:**

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:**

**Rationale:**

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:**

**Rationale:**



.. _Other Findings and Identified Issues:

Other Findings and Identified Issues
====================================

During the existance of this working group, numerous items were identified as problematic and needing to be addressed but either were not well fit to a charge question or fell out of the scope of the charge.
This section contains information regarding numerous issues which were identified and require attention.

- Lack of definition regarding degraded mode(s)
