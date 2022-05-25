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
The charge was developed based on the findings during the Missing Functionality Workshop that took place February 2-3, 2022.
The charge addresses three specific Jira tickets developed from that meeting: `SITCOM-174`_, `SITCOM-173`_, and `SITCOM-180`_.
These are referenced throughout the report.

Much of this report builds off the findings and recommendatations of the first `FAFF report`_. 
It is expected that the readers are already familiar with the findings and recommendatations.

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

      - Use-cases should be complete, including which inputs are required and from where they will originate (e.g. SAL Script, EFD, LFA, external source), desired manipulations, logic-based operations/calculations, and if/how the desired artefacts are presented to the user (e.g. display images and/or graphs).
  

Numerous use cases were developed to capture the needed functionalities and assist in developing a common understanding of what is expected in each scenario.
Each of the use cases follow a standardized `template <https://confluence.lsstcorp.org/display/LSSTCOM/On-the-fly+Analysis+Use-Case+Template>`_ which differs slightly from that which was used in the first FAFF charge.
One major difference is that nearly all use cases made the assumption that there would be a process which created data products based on observatory telemetry and camera images that would be used as the basis for displaying information.
Rather than each use-case stating the needed data prodcuts and explaining their generation mechanism, we created a single use-case that documents the needed values and timescales for their creation.
We call this processing step `Rapid Analysis <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`, and the data products are listed explicitly, as requested in charge question 2.

The remaining use-cases for FAFF2 can be found at the bottom of the page `on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Use-Cases>`_ and are referenced throughout the remainder of this report.


.. _Deliverable 2:

Deliverable 2: Rapid Analysis Calculated Metrics
------------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   2. (`SITCOM-180`_, `SITCOM-173`_) Define which metrics, analyses and artifacts must be calculated and on what timescale they must be evaluated and reported to support commissioning/operations. 
   
      This is to evaluate if a "rapid processing" of data is required, what specific calulations are required.
      This list should include the relevant camera specific calculations (which are currently performed by the EO testing data reduction).
      This is expected to inform the answer to the next charge task.


Numerous calculations are required to evaluate camera and system health and performance on rapid timescales.
The data products discussed in this section are limited to scalars and/or arrays and does *not* include the generation of plots and/or figures.things like plots/figures.
The high majority of the data needed on rapid timescales is consistent with a pared down version of the data products produced by the single-frame-processing (SFP)framework.
A small number of additional values are also required, but can be quickly derived from the SFP results.

Based on the committee's experience commissioning previous telescopes, instruments and surveys, three different timescales for data interaction were identified as being critical to successful commissioning:

<30 seconds
^^^^^^^^^^^
This is the timescale where data feedback must be made available that could influence the next activity, configuration, or exposure. Examples include displaying of images and fundamental health metrics. Also engineering tasks where corrections or instrument setups may be changed, a new image is taken, and it is useful to know if the changes impacted the image as expected.

The camera commissioning cluster is the first significant computing infrastructure to have access to the pixel data.
This is where the Camera Visualization Tool (CVT) is to be run such that users can see the images with the lowest possible latency.
It is also where the camera system conducts low-level measurements to determine camera health, such as median and standard deviation of the overscan regions.
This is then used to help inform the camera health displays discussed in Use-Case FIXME.
Further details regarding use of the commissioning cluster as discussed in `Deliverable 5: Computing Resources (Clusters)`_.


~60 seconds
^^^^^^^^^^^
This timescale is useful when examine trending or slowly varying effects, particularily in things like image quality or transparency.
It is a timescale where people are closely watching, but not necessarily immediately reacting.
The addition of this category was to provide flexibility in implementation as it may be such that the prioritization of metrics can be performed.

However, it is imperative that the rapid analysis framework be able to keep up with the rate of images being acquired; where that rate is governed by the survey strategy visit duration (`FAFF-REQ-XXX1`_).
In the case of taking two 15 second snaps, it is expected that the analysis would be done on the combined images.
The data products for the <30 and 60 second timscales are described in the Outputs section of the `Rapid Analysis Use-case on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_.


12-24 24-hours
^^^^^^^^^^^^^^
This timescale is important for more general commissioning activities and performance assessment that could impact observations taken in the next or subsequent nights.
Over this timescale, a full DRP single-frame-processing pipeline needs to have been run (`FAFF-REQ-XXX2`_).
This must include the additional values that are calculated in the Rapid Analysis Framework, which can be easily added to the SFP pipeline.
This enables a more detailed and higher-confidence data quality evaluation to be performed, including correlation with telemetry, environmental conditions, and previous conditions and/or observations.
It also allows the teams to begin determining which subsets of data should be used to construct coadds/templates, begin SV analyses, and ultimately maximize the number of human brain cycles looking at the data.
It is fully expected that this dataset will be superseded by a subsequent DRP campaign to enforce that all the data is processed in a homogeneous way with best performing configuration of the science pipelines.

It is not required that the full SVP processing be done in Chile, in fact, it may be *preferable* to perform this processing at the USDF as many of the science verification tasks are planned to be performed there as well.
It also ensures that a minimum number of uses are connecting to Chile to perform their analysis.
This is especially important if connctions would be required to the summit instance.


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
- Webpages
- Trending plots (see also `Deliverable 4`_ for discussion of scalar fields as a function of a 3rd axis)

Useful to group into aggregated (binned) and non aggregated (unbinned) metrics.

- Binned: aggregated values that are pre-computed on a certain spatial scale (e.g. an amp/detector/raft) data where the scaling can be changed and could be modified to varying scales
- Unbinned: Value per source (e.g. photometry measurement at each previous visit).
 Depending on the case, a slider could be present to adjust the scaling on-the-fly

.. _Deliverable 4:

Deliverable 4: Required Non-Scalar Metrics
------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  4. (`SITCOM-180`_) Provide a list of required non-scalar metrics are required and cannot be computed with faro. 
     Suggest a mechanism (work flow) to perform the measurement, document the finding, evaluate any trend (if applicable), then present it to the stakeholders.
    

.. related to https://confluence.lsstcorp.org/display/LSSTCOM/Displaying+scalar+fields+as+a+function+of+other+parameters
This is trending, but not only in time. 
For example, it could be a scalar field's metric as a function of a third axis.
Examples include: PSF shape over the field as a fxn of elevation, Sky transparency as a function of time etc.

Faro computes single valued (scalar) metrics and compares against an expected value or range (e.g. a sigma or mean).
It was decided that there is not a use-case where we are unable represent a scalar field with respect to a third axis (e.g. time/elevation etc) as a single valued metric (e.g. a mean, or stddev). 
However, representing a field as a single metric can hide underlying systematics, such as having only one side of the field having an effect, which is not noticed when looking only at a single number representing the entire field.
For this reason, and for the more general reason of needing the ability to dig into the data when a metric is not within the expected range, it is required to have the ability to view and reproduce the data that went into calculating the faro metric. 
`FAFF-REQ-XYZZ`_ has been created to capture this functionality.

When diagnosing the data, the plots and investigations can be time consuming to code up.
Because in all FAFF related use-cases we are dealing with aggregated data, it would be useful to generate a generic application, most likely in Bokeh, that can present both sky and focal plane aggregated data as a function of a 3rd axis of interest.
This would should be carried out with the DM DRP team which also need the same functionality and should therefore use the same toolset.
Naturally, people should be able to fork and customize the app for specific implementations if required, although we expect that the general set of functionalities will be sufficient to support the majority of use-cases.

Functionality of the tool could include:
- Ability to flip through a 2-d data cube as a movie
- Click on a given amp and have a plot of the value versus time, with the expecation value of the metric over plotted etc.
- Ability to show sky maps as a function of time, and adjust the binning on-the-fly
- Capable of mining the appropriate data given the specific faro metric (including timestamp etc)

Lastly, it is recognized that the DM DRP team also needs to interact with non-aggregated data, this is outside the scope of FAFF, however, adopting a common toolset, or one that is based off the tooling being discussed here is recommended.



.. _Deliverable 5:

Deliverable 5: Computing Resources (Clusters)
---------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  5. (`SITCOM-174`_) Using the responses to questions 1-4, propose a management & maintenance structure for the Camera Diagnostic & Commissioning Clusters.
     
     This includes identifying what processes require specific hardware and/or infrastructure, identifying the more generalized analyses that may benefit from a common infrastructure, and evaluating possible solutions that can ease duplication of effort.
    

- The most significant benefit of the Camera Diagnostic Cluster is that it has immediate access to the images. 
  Secondly, is it located on the summit and can therefore continue to be used in the event of a network outage to the base. 
  This means it is able to make display(s), such as what is done with the CVT, and perform calculations prior to the image being (potentially transferred) and subsequently ingested by the butler prior to performing calculations with the DM pipeline(s).

- Rapid analysis is envisioned to run at the base, where there is significantly more computing power and storage
- Rapid Analysis will use DM pipelines
- Camera diagnostic cluster uses a simplified set of tools to perform rapid yet rudamentary on-the-fly calculations, all in camera space.
  Using the DM toolset, although useful, would add significant complexity, specifically in regards to maintenance/updates, that would go largely unused if the desire was only to replace the values being calculated now during EO testing.

Unfortunately, the definition of what needs to be calculated on the summit to support operations is very heavily tied to the concept of "Degraded mode," which is currently not sufficiently defined to draw a single conclusion.
For the sake of this charge question, the functionality of critical importance is the connection to the base, and therefore we can address this charge question by considering two separate scenarios:

1. Degraded mode means the observatory is able to continue standard survey operations, but at an increased risk of obtaining poor data because of a disconnection from the rapid analysis framework.
2. Degraded mode requires the observatory continue to receive output from the rapid analysis framework to support operations, scheduler input, or potentially QA analyses etc. 

In the case of scenario 1, in the event of a network outage, the summit-based diagnostic cluster will run the CVT, and continue to perform the low-level calculations that will go into the camera's database and the EFD.
Observers will see the images being output and be able to visually evaluate performance.
Because no rapid-analysis support will be available from the base, any (non-AOS) image-based calculations will not be performed and therefore it is possible that certain enginnering tests will not be able to be performed, and (potentially) certain inputs to the scheduler may not arrive.

In scenario 2, where a subset of rapid analysis is required (which we refer to as rapid-analysis-critical) to remain functional in the event of an outage, this requires a very significant increase in functionality.
- DM tooling must be installed and maintained on the diagnostic cluster
- Rapid-analysis-critical must be developed and deployed, with the ability to only focus on a subset of detectors, and/or metrics
- The database containing the output must be hosted on the summit, then replicated outwards

Note that in the event of scenario 2, certain tasks will still be limited as the computer power is significantly lower which will result in the computations taking longer, and being only a subset.

Regardless of how degraded mode is ultimately defined, this committee recommends a step-wise approach where the infrastructure for scenario 1 gets implemented prior to significant effort being put into scenario 2, if deemed appropriate.

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

Tiago working on a proposed high-level design for this is in consultation with Angelo.
The proposed implementation is described in `a technote <tstn-034.lsst.io>`_.


.. _Deliverable 8:

Deliverable 8: Training
-----------------------
.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  8. Design user-level training bootcamps and materials, aimed at the level of an in-kind contributor.
     
     These bootcamps will be used as the initial training materials.
     It is expected that In-kind contributors and/or other delegates can augment the content, provide improvements, and eventually take over some of the training.

List of possible trainings:

- Creation of a job that spawns a calculation, creates an artifact, displays the artifact, and alerts a user
- Creation of a Bokeh App to be used during the night based on already available data

- Using the CVT (as a fxn of location) - this would be a basic guide and not a bootcamp. 
   - Basic operations for viewing images
   - Interactions with DM tools/features such as source detections

- Possibly out of FAFF scope - but a "bootcamp" about operationing the telescope via LOVE, and writing SAL scripts.

Building a "pipetask" is out of scope for FAFF.


.. _Deliverable 9:

Deliverable 9: Task Prioritization
----------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  9. A prioritized list of tasks to build-out the new functionalities with recommended end-dates. 
     
     Where possible, these dates shall correspond to integration milestones.


The following tasks are highly parralelizable. 

#. Define computing resources strategy (Deliverable 6)
   - need to know if antu will go to the summit 
#. Get basic camera diagnostics running at the summit.
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
#. 



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

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** All processed data and artifacts shall be referenced from a single source, as viewed from the user.

**Rationale:** Users will need to access EFD data, rapid processing data, and all generated artifacts in the same manner. 
They need not be pre-occupied with where the data exists and why. 
This requirement does not specify everything must be stored in a single database, although it may be a solution.
It is also acceptable that a query returns a link to a file in the LFA.

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** The rapid analysis processed data and artifacts must be accessible from the major data processing facilities (e.g. Summit, base, USDF).

**Rationale:** This will probably require replication of the data, analogous to the EFD.


FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** Rapid analysis shall produce data products that are not critical to operations/commissioning.

**Rationale:** The telescope need not stop observing if the rapid analysis fails, however, it is expected that functionality may be reduced and/or the planned observations/activities may change.


FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** Rapid analysis data processing (and storage) shall only be run once.

**Rationale:** This is a one-off on-the-fly analysis.
Data products, even if incorrect, will remain as such.
This is intentional to keep a record of what was available to the user (and/or scheduler) at a later time.
Because rapid analysis is not re-run, no versioning or relationships to other calculated results in the future need to be supported.


FAFF-REQ-XXX2
^^^^^^^^^^^^^
**Specification:** A full single-frame-processing shall be run on images within 24 hours of observation.

**Rationale:** Ideally this would be done in less than 12 hours, so people could look at it before the next night's observation, although this is a stretch goal.
This data uses the most recent (best) science pipelines and produces the highest quality data products that are used for science verification tasks.


FAFF-REQ-XXXX
^^^^^^^^^^^^^

PI: I'm not sure this is a necessary requirement. 
Also, if the rapid analysis has something special it calculates, how can it be recalculated? 

**Specification:** Observers shall be able to run instances of single-frame-processing manually to support commissioning.

**Rationale:** If rapid analyis fails, then users will need the capability to re-run the analyses.
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