.. _preparing_data__streaming__flows:

Flows
=====

.. include:: ../shared/flows/introduction.rst

The SDK provides the following functionality to interact with streaming flows:
    * Creating a flow
    * Retrieving flows
    * Editing a flow
    * Updating a flow
    * Duplicating a flow
    * Deleting a flow
    * Exporting flows
    * Importing flows
    * Validating a flow
    * Previewing a flow
    * Handling error records

.. _preparing_data__streaming__flows__prerequisites:

Prerequisites
~~~~~~~~~~~~~

To create a flow using the SDK, please make sure you have done the following steps:
    * :ref:`Created a project<projects__projects__creating_a_project>`.
    * :ref:`Created an environment in the project <projects__environments__creating_an_environment>`.
    * :ref:`Installed an engine in your environment<projects__environments__retrieving_install_command>`.

.. note::
    Currently, the SDK only supports creating a flow with an engine installed.

.. _preparing_data__streaming__flows__creating_a_flow:

Creating a Flow
~~~~~~~~~~~~~~~

In the UI, you can create a flow by navigating to **Assets -> New asset -> Create a flow**.

.. warning::
    Creating a streaming flow in the UI requires setting an environment

.. image:: /_static/images/flows/create_streaming_flow.png
    :alt: Screenshot of the flow creation page.
    :align: center
    :width: 100%

In the SDK, you can create a flow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_flow>` method.
You are required to supply a ``name`` and ``environment`` parameters.
On how to create an environment check :ref:`creating an environment <projects__environments__creating_an_environment>`.
You can also provide an optional ``description``.
You don't have to specify a ``flow_type`` as its default value is ``streaming``.
This method returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` instance.

.. code-block:: python

    >>> new_flow = project.create_flow(name='My streaming flow', description='optional description', environment=environment)
    >>> new_flow
    StreamingFlow(name='My streaming flow', description='optional description', flow_id=..., engine_version=...)

.. _preparing_data__streaming__flows__retrieving_a_flow:

Retrieving Flows
~~~~~~~~~~~~~~~~

Flows can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.flows <ibm_watsonx_data_integration.cpd_models.project_model.Project.flows>` property.
If you wish to only retrieve streaming flows use the :py:meth:`Project.flows.get_all() <ibm_watsonx_data_integration.common.models.CollectionModel.get_all>` method to filter by ``flow_type``.
You can also retrieve a single flow using the :py:meth:`Project.flows.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method which requires the ``flow_id`` parameter.

.. code-block:: python

    >>> project.flows  # a list of all flows
    [...StreamingFlow(name='My streaming flow', description='optional description', ...)...BatchFlow(..., name='My batch flow', description='optional description', ...)...]
    >>> project.flows.get_all(flow_type='streaming')
    [...StreamingFlow(name='My streaming flow', description='optional description', ...)...]
    >>> project.flows.get(name='My streaming flow')
    StreamingFlow(name='My streaming flow', description='optional description', ...)

.. _preparing_data__streaming__flows__editing_a_flow:

Editing a Flow
~~~~~~~~~~~~~~

.. include:: ../shared/flows/editing_flow.rst

On top of that streaming flows have many properties related to pipeline, engines, etc., you can edit a streaming flow's configuration through the :py:attr:`Flow.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.configuration>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object which encapsulates a flow's configuration.
You can print out the configuration and edit it similarly to a :py:class:`dict`.

.. code-block:: python

    >>> new_flow.configuration['retry_pipeline_on_error']
    True
    >>> new_flow.configuration['retry_pipeline_on_error'] = False


.. _preparing_data__streaming__flows__updating_a_flow:

Updating a Flow
~~~~~~~~~~~~~~~

.. include:: ../shared/flows/updating_flow.rst

.. _preparing_data__streaming__flows__duplicating_a_flow:

Duplicating a Flow
~~~~~~~~~~~~~~~~~~

.. include:: ../shared/flows/duplicating_flow.rst

.. _preparing_data__streaming__flows__deleting_a_flow:

Deleting a Flow
~~~~~~~~~~~~~~~

.. include:: ../shared/flows/deleting_flow.rst

.. _preparing_data__streaming__flows__validating_a_flow:

Validating a Flow
~~~~~~~~~~~~~~~~~

In the UI, you can validate a streaming flow by making changes to the flow and hitting the 'Validate' icon to validate the flow.

.. image:: /_static/images/flows/validate_flow_button.png
   :alt: Screenshot of the validate button.
   :align: center
   :width: 100%

To validate a streaming flow via the SDK, you need to update a flow, and then call the :py:meth:`StreamingFlow.validate() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.validate>` method.
This will return a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.ValidationResult` object.
This object contains ``issues`` attribute that has a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.FlowValidationError` instances if there are any errors.

.. code-block:: python

    >>> new_flow.add_stage('Trash')
    Trash_01(stage_name='Trash 1')
    >>> project.update_flow(new_flow)
    <Response [200]>
    >>> new_flow.validate()
    ValidationResult(success=False, issues=[FlowValidationError(type='stageIssues', instanceName='Trash_01', humanReadableMessage='The first stage must be an origin'), FlowValidationError(type='stageIssues', instanceName='Trash_01', humanReadableMessage='Target must have input streams')], message='Validation Failed')

.. _preparing_data__streaming__flows__previewing_a_flow:

Previewing a flow
~~~~~~~~~~~~~~~~~

In the UI, you can preview a flow by hitting the 'Preview' icon.

.. image:: /_static/images/flows/preview_flow_button.png
   :alt: Screenshot of the preview button.
   :align: center
   :width: 100%

To preview a flow via the SDK, you need to call :py:meth:`StreamingFlow.preview() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.preview>` method.
This will return a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.PreviewStage` instances.
Each PreviewStage provides access to its :py:attr:`input <ibm_watsonx_data_integration.services.streamsets.models.flow_model.PreviewStage.input>` and :py:attr:`output <ibm_watsonx_data_integration.services.streamsets.models.flow_model.PreviewStage.output>` properties, which contain the input and output data for that stage.


.. code-block:: python

    >>> preview = flow.preview()
    >>> preview
    [PreviewStage(instance_name='DevRawDataSource_01'), PreviewStage(instance_name='Trash_01')]
    >>> dev_raw_data_preview, trash_preview = preview
    >>> dev_raw_data_preview.input
    >>> dev_raw_data_preview.output
    [('abc', 'xyz', 'lmn')]

.. _preparing_data__flows__exporting_flows:

.. include:: ../shared/flows/exporting_flows.rst

.. _preparing_data__flows__importing_flows:

Importing Flows
~~~~~~~~~~~~~~~

To import a flow via the SDK, you need to call the :py:meth:`Project.import_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.import_flows>` method and you
must specify the ``source`` parameter, which is the Path to the zip file containing the json(s) of the streaming flow(s) to be imported and the ``type`` parameter, which should be
``'streaming'``.

You must also set the ``conflict_resolution`` parameter which determines how to handle an attempted import of a duplicate flow which already exists in the project. The options for ``conflict_resolution`` are listed below:
    * ``'skip'`` – skip this particular flow and move to the next flow to be imported.
    * ``'replace'`` - replace the existing flow with the flow to be imported.
    * ``'rename'`` - keep the existing flow and rename the flow that is being imported.

The function will return the imported :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` objects
or it will return a list of the imported :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` objects.

.. skip: start 'Import Flows API endpoint not on Prod yet'

.. code-block:: python

    >>> project.import_flows(type='streaming', source='flows_to_import.zip', conflict_resolution='skip')
    StreamingFlow(name='dummy_flow', description='dummy_description', flow_id='...', engine_version='...')

.. skip: end

.. _preparing_data__streaming__flows__handling_error_records:

Handling Error Records
~~~~~~~~~~~~~~~~~~~~~~

To edit error record handling on the UI, click the gear icon on the top-right of the screen in a flow's edit page.

.. image:: /_static/images/flows/save_flow_button.png
   :alt: Screenshot of opening a flow settings.
   :align: center
   :width: 100%

This opens a new pop-up window with a tab for ``Error records`` on the left. This will let you adjust the error record handling for the flow.

.. image:: /_static/images/flows/flow_settings.png
   :alt: Screenshot of flow settings.
   :align: center
   :width: 100%

This page lets you change how error records are handled by policy and which stage should handle them.

The possible options for error record policy are ``Original record as it was generated by the origin`` and
``Record as it was seen by the stage that sent it to error stream``.
In the SDK, these equate to ``ORIGINAL_RECORD`` and ``STAGE_RECORD``.

This can be updated in a flow's configuration.

.. code-block:: python

    >>> new_flow.configuration['error_record_policy']
    'ORIGINAL_RECORD'
    >>> new_flow.configuration['error_record_policy'] = 'STAGE_RECORD'


To change the error record stage, you can call :py:meth:`StreamingFlow.set_error_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.set_error_stage>` method.
You need to pass either the ``label`` or the ``name`` of the new error stage, you can also optionally pass in the new stage's ``library``.

.. note::

    All error stages other than ``Discard`` will have configuration options for you to customize your experience.

.. code-block:: python

    >>> write_to_file = new_flow.set_error_stage('Write to File')
    >>> write_to_file.configuration['directory'] = '/path/to/some/directory'

You can view the current error stage for a flow at any point using the :py:attr:`StreamingFlow.error_stage <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.error_stage>` property.

.. code-block:: python

    >>> new_flow.error_stage
    WritetoFile_ErrorStage(stage_name='Error Records - Write to File')
