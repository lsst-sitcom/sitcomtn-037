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

Abstract
========

The First-Look Analysis and Feedback Functionality Breakout Group `charge`_ #2 details questions regarding the displays and functionalities required to perform continuous nightly commissioning activities. The focus is on deriving the required functionalities to perform inspection and analyses within ~24-hours of taking the data. This document details the answers to the charge questions and other findings.

Executive Summary
=================

This is the summary.


Introduction
============

This is the intro.

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
  
Use-cases for FAFF2 can be found at the bottom of the page `on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Use-Cases>`_.


.. _Deliverable 2:

Deliverable 2: Rapid Analysis Calculated Metrics
------------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   2. (`SITCOM-180`_, `SITCOM-173`_) Define which metrics, analyses and artifacts must be calculated and on what timescale they must be evaluated and reported to support commissioning/operations. 
   
      This is to evaluate if a "rapid processing" of data is required, what specific calulations are required.
      This list should include the relevant camera specific calculations (which are currently performed by the EO testing data reduction).
      This is expected to inform the answer to the next charge task.

  
This is described in the Outputs section of the `Rapid Analysis Use-case on confluence <https://confluence.lsstcorp.org/display/LSSTCOM/Rapid+Analysis+Use-Case>`_.

This will be limited to scalars/arrays (essentially the data for plots) and will *not* include things like plots.

Derived Requirements
^^^^^^^^^^^^^^^^^^^^



.. _Deliverable 3:

Deliverable 3: Interacting with Rapid Analysis Metrics
------------------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

   1. (`SITCOM-174`_, `SITCOM-173`_) Define how users will interact with each aspect of the previously listed metrics, analyses and artifacts; classify them indicating where can could calculated.
      
      This includes tasks defined for the catcher, OCPS jobs, AuxTel/ComCam/LSSTCam processing, and the rendez-vous of data from multiple sources (DIMM, all-sky etc).

This is:

- camera visualization health tool(s).
- Extended functionality of the CVT.
- Bokeh Apps 
- Webpages
- Trending plots?

.. _Deliverable 4:

Deliverable 4: Required Non-Scalar Metrics
------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  4. (`SITCOM-180`_) Provide a list of required non-scalar metrics are required and cannot be computed with faro. 
     Suggest a mechanism (work flow) to perform the measurement, document the finding, evaluate any trend (if applicable), then present it to the stakeholders.
    

This is trending, but not only in time. 
For example, it could be a metric's evolution with airmass.

This is Robert's use-case.  

.. _Deliverable 5:

Deliverable 5: Computing Resources (Clusters)
---------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  5. (`SITCOM-174`_) Using the responses to questions 1-4, propose a management & maintenance structure for the Camera Diagnostic & Commissioning Clusters.
     
     This includes identifying what processes require specific hardware and/or infrastructure, identifying the more generalized analyses that may benefit from a common infrastructure, and evaluating possible solutions that can ease duplication of effort.
    

Not yet ready to have this conversation, it requires completion of the above.

.. _Deliverable 6:

Deliverable 6: Computing Resources (Clusters)
---------------------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  6. Develop a plan and scope estimate to expand the Camera Visualization Tool to support the full commissioning effort.
     
     This includes identifying libraries/packages/dependencies that require improvements (e.g. Seadragon) and fully scoping what is required to implement the tool with DM tooling such as the Butler. 
     The scope estimate may propose the use of in-kind contribution(s) to this effort if and where applicable.

This is Tony and Gregory to come up with a first crack at this. 
Tony already has a document with questions/issues; awaiting Gregory to discuss

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

.. _Deliverable 8:

Deliverable 8: Training
-----------------------
.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  8. Design user-level training bootcamps and materials, aimed at the level of an in-kind contributor.
     
     These bootcamps will be used as the initial training materials.
     It is expected that In-kind contributors and/or other delegates can augment the content, provide improvements, and eventually take over some of the training.

List of possible trainings:

- Creation of a Bokeh App to be used during the night based on already available data
- Creation of a job that spawns a calculation, creates an artifact, and alerts a user
- Using the CVT (as a fxn of location)

   - Basic operations for viewing images
   - Interactions with DM tools/features such as source detections


.. _Deliverable 9:

Deliverable 9: Task Prioritization
----------------------------------

.. note:: 

   The deliverable description from the `charge`_ has been directly copied here to ease readability.

  9. A prioritized list of tasks to build-out the new functionalities with recommended end-dates. 
     
     Where possible, these dates shall correspond to integration milestones.

Current thinking:

1. Define computing resources strategy (Deliverable 6)
2. Get catcher deployed (needed for telescope engineering). Camera can continue to use it's tooling. 
3. Get Rapid Analysis Framework deployed
4. Get database deployed/operational 
5. Merge tooling/toolsets to become a unified Framework 
6. Develop training examples (actually performed in conjunction with the previous)


.. _Derived Requiremends:

Generated Requirements
======================

Based upon the above use-cases, numerous requirements on to-be-designed and implemented systems have been derived.
This section captures these and roughly organizes them.

Processing
----------

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** All processed data and artifacts shall be referenced from a single source, as viewed from the user.

**Rationale:** Users will need to access EFD data, rapid processing data, and all generated artifacts in the same manner. 
They need not be pre-occupied with where the data exists and why. 
This requirement does not specify everything must be stored in a single database, although it may be a solution.
It is also acceptable that a query returns a link to a file in the LFA.

FAFF-REQ-XXXX
^^^^^^^^^^^^^
**Specification:** The processed data and artifacts must be accessible from the major data processing facilities (e.g. Summit, base, USDF).

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

FAFF-REQ-XXXX
^^^^^^^^^^^^^

PI: I'm not sure this is a necessary requirement. 
Also, if the rapid analysis has something special it calculates, how can it be recalculated? 

**Specification:** Observers must be able to run instances of single-frame-processing manually to support commissioning.

**Rationale:** If rapid analyis fails, then users will need the capability to re-run the analyses.
This is expected to be done either at the USDF or on the commissioning cluster.
It is expected that this is essentially a single line of code, but will require training.


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