.. _overview__release_notes:

What's new
==========

Below you will find release notes for each version of the ``ibm-watsonx-data-integration`` SDK as well as highlights of major features and/or fixes included in each release.

1.3.1 (June 2026)
------------------------
* Restored ``Stage.stage_name`` property with deprecation warning. Will be removed in next major release. Use ``Stage.name`` instead.


1.3.0 (May 2026)
------------------------
* Added JobRun.logs and JobRun.metrics support for streaming jobs.

  See :ref:`JobRun Logs <projects__jobs__run_logs>`.

* Fixed project asset ownership validation.

  See :ref:`Projects  <projects__projects>`

* Python code generator enhancements and improvements.

  See :ref:`Code Generation (Batch) <getting_started_and_tutorials__code_generation_batch>`,
  :ref:`Code Generation (Streaming) <getting_started_and_tutorials__code_generation_streaming>`,
  and :ref:`Code Generation for Connections <getting_started_and_tutorials__code_generation_connections>`.

* Quality of life enhancements and bug fixes.


1.2.0 (April 2026)
------------------------
* Added support for WebSocket tunneling for secure communication.

  See :ref:`Platform initialization <getting_started_and_tutorials__platform>` for connection configuration.

* Added support for job parameters to enable dynamic job configuration.

  See :ref:`Creating a Job <projects__jobs>` and :ref:`Parameter Sets <preparing_data__parameter_sets>`.

* Added support for annotations and comments in batch flows for improved documentation.

  See :ref:`Batch Flows <preparing_data__batch__flows>`.

* Added support for data definitions in batch flows for better schema management.

  See :ref:`Data Definitions <preparing_data__data_definitions>`.

* Added stage discovery functionality for both streaming and batch flows.

  See :ref:`Batch Stages <preparing_data__batch_stages>` and :ref:`Streaming Stages <preparing_data__streaming_stages>`.

* Flow topology enhancements.

  See :ref:`Code Generation (Batch) <getting_started_and_tutorials__code_generation_batch>`.

* Python code generator enhancements and improvements.

  See :ref:`Code Generation (Batch) <getting_started_and_tutorials__code_generation_batch>`,
  :ref:`Code Generation (Streaming) <getting_started_and_tutorials__code_generation_streaming>`,
  and :ref:`Code Generation for Connections <getting_started_and_tutorials__code_generation_connections>`.

* Quality of life improvements and bug fixes.


1.1.0 (February 2026)
------------------------
* Added Python 3.14 support.

* Added support for proxies, custom SSL certificates, and disabling SSL verification.

  See :ref:`Platform initialization <getting_started_and_tutorials__platform>`.

* Fixed UI topology not persisting when editing flows.

  See :ref:`Batch Flows <preparing_data__batch__flows>` and :ref:`Streaming Flows <preparing_data__streaming__flows>`.

* Python code generator improvements.

  See :ref:`Code Generation (Batch) <getting_started_and_tutorials__code_generation_batch>` and
  :ref:`Code Generation (Streaming) <getting_started_and_tutorials__code_generation_streaming>`.

* Improved UX for working with batch stages and links.

  See :ref:`Batch Stages <preparing_data__batch_stages>`.

* Added ability to edit job configuration.

  See :ref:`Jobs <projects__jobs>`.

* Bug fixes and improvements.

* Minor updates for enhanced SDK functionality.


1.0.0 (December 2025)
------------------------
* Added support for retrieving, creating, updating, and deleting ParameterSets.

  See :ref:`Parameter Sets <preparing_data__parameter_sets>`.

* Added support for creating, updating, and deleting ValueSets.

  See :ref:`Parameter Sets <preparing_data__parameter_sets>`.

* Added support for retrieving, creating, updating, and deleting batch flow Subflows.

  See :ref:`Subflows <preparing_data__subflows>`.

* Added support for BatchFlow runtime settings.

  See :ref:`Batch Flows <preparing_data__batch__flows>`.

* Added support for BatchFlow compilation modes (ETL/ELT/TETL).

  See :ref:`Compilation Modes <preparing_data__batch__compilation_mode>`.

* Added support for generating Python code for StreamingFlows and Connections.

  See :ref:`Code Generation (Streaming) <getting_started_and_tutorials__code_generation_streaming>` and
  :ref:`Code Generation for Connections <getting_started_and_tutorials__code_generation_connections>`.

* Added support for BearerToken, ZenApiKeyAuthenticator, and ICP4DAuthenticator authentication methods.

  See :ref:`Authentication <getting_started_and_tutorials__authentication>`.

* Added support for retrieving, adding, removing, and updating ProjectCollaborators.

  See :ref:`Project Collaborators <projects__project_collaborators>`.

* Added support for importing and exporting StreamingFlows.

  See :ref:`Streaming Flows <preparing_data__streaming__flows>`.

* Added support for IBM Cloud Pak for Data on-premises.

  See :ref:`Authentication <getting_started_and_tutorials__authentication>` and :ref:`Platform initialization <getting_started_and_tutorials__platform>`.

* Bug fixes and improvements.

* Minor updates for enhanced SDK functionality.


1.0.0b (September 2025)
-----------------------
* Added support for retrieving, adding, updating, and removing batch flows.

  See :ref:`Batch Flows <preparing_data__batch__flows>`.

* Added support for executing batch flows within a job.

  See :ref:`Jobs <projects__jobs>`.

* Added Python generator support for batch and streaming flows.

  See :ref:`Code Generation (Batch) <getting_started_and_tutorials__code_generation_batch>` and
  :ref:`Code Generation (Streaming) <getting_started_and_tutorials__code_generation_streaming>`.

* Added support for engineless streaming flows.

  See :ref:`Streaming Flows <preparing_data__streaming__flows>`.

* Added support for streaming job offsets.

  See :ref:`Jobs <projects__jobs>`.

* Added support for validating streaming flows.

  See :ref:`Streaming Flows <preparing_data__streaming__flows>`.

* Bug fixes and improvements.

* Minor updates for enhanced SDK functionality.


0.0.2b (August 2025)
--------------------
* Added support for retrieving, adding, and removing members from AccessGroups.

  See :ref:`Access Control <access_control>`.

* Engine and Environment API clients & model moved under streaming flows service.

  See :ref:`Streaming Engines <projects__engines>` and :ref:`Streaming Environments <projects__environments_streaming>`.

* Access pattern updates for all resource IDs.

* Bug fixes across the SDK.

* Minor updates for enhanced SDK functionality.


0.0.1b (June 2025)
------------------
* Added support for retrieving, creating, updating, deleting and running Jobs.

  See :ref:`Jobs <projects__jobs>`.

* Added support for retrieving, creating, updating, and deleting Projects.

  See :ref:`Projects <projects__projects>`.

* Added support for retrieving, creating, updating, and deleting StreamingFlows.

  See :ref:`Streaming Flows <preparing_data__streaming__flows>`.

* Added support for retrieving, creating, updating, and deleting Connections.

  See :ref:`Connections <preparing_data__connections>`.

* Added support for retrieving, creating, updating, and deleting StreamingFlow Environments.

  See :ref:`Streaming Environments <projects__environments_streaming>`.

* Added support for retrieving, creating, updating, and deleting AccessGroups.

  See :ref:`Access Control <access_control>`.

* Added support for retrieving, creating, updating, and deleting Roles.

  See :ref:`Access Control <access_control>`.

* Added support for retrieving Users & Accounts.

  See :ref:`Access Control <access_control>`.

* Added the ability to retrieve the install script for StreamingFlow Engines.

  See :ref:`Streaming Engines <projects__engines>`.
