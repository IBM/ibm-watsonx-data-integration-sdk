.. _preparing_data__flows:

Flows
=====
|

A flow is an object used for storing the execution flow of a data pipeline.
A flow is comprised of multiple stages with each stage defining how data is handled in that part of the execution flow.

The SDK provides the following functionality to interact with flows:
    * Creating a flow
    * Retrieving flows
    * Editing a flow
    * Updating a flow
    * Duplicating a flow
    * Deleting a flow
    * Validating a flow
    * Handling error records

.. _preparing_data__flows__prerequisites:

Prerequisites
~~~~~~~~~~~~~

To create a flow using the SDK, please make sure you have done the following steps:
    * :ref:`Created a project<projects__projects__creating_a_project>`.
    * :ref:`Created an environment in the project <projects__environments__creating_an_environment>`.
    * :ref:`Installed an engine in your environment<projects__environments__retrieving_install_command>`.

.. note::
    Currently, the SDK only supports creating a flow with an engine installed.

.. _preparing_data__flows__creating_a_flow:

Creating a Flow
~~~~~~~~~~~~~~~

In the UI, you can create a flow by navigating to **Assets -> New asset -> Create a flow**.

In the SDK, you can create a flow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_flow>` method.
You are required to supply a ``name`` parameter and an optional ``description`` parameter.
You should also specify a ``flow_type`` parameter which decides what type of flow you will get.
    * If ``flow_type='streamsets'``, then you get a streaming flow of the :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow` instance.
        * If no ``flow_type`` is specified, this type of flow is returned by default.

        .. warning::
            Note that creating streaming flows in the UI requires setting an environment:

            .. image:: ../../_static/images/flows/create_flow.png
                :alt: Screenshot of the flow creation page.
                :align: center
                :width: 100%

            Therefore, an ``environment`` parameter is also necessary for streaming flows.
    * If ``flow_type='datastage'``, then you get a batch flow of the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.datastage_flow.DataStageFlow` instance.

This method will return a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` of the specified flow subclass according to ``flow_type``.

.. code-block:: python

    >>> new_flow = project.create_flow(name='My first flow', description='optional description', environment=environment)
    >>> new_flow
    StreamsetsFlow(name='My first flow', description='optional description', flow_id=..., engine_version=...)
    >>> new_flow_2 = project.create_flow(name='My second flow', description='optional description', environment=None, flow_type='datastage')
    >>> new_flow_2
    DataStageFlow(..., name='My second flow', description='optional description', flow_id=...)

.. _preparing_data__flows__retrieving_a_flow:

Retrieving Flows
~~~~~~~~~~~~~~~~

Flows can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.flows <ibm_watsonx_data_integration.cpd_models.project_model.Project.flows>` property.
You can also retrieve a single flow using the :py:meth:`Project.flows.get() <ibm_watsonx_data_integration.cpd_models.flows_model.Flows.get>` method
which requires the ``flow_id`` parameter.

.. code-block:: python

    >>> project.flows  # a list of all the flows
    [...StreamsetsFlow(name='My first flow', description='optional description', ...)...DataStageFlow(..., name='My second flow', description='optional description', ...)...]

    >>> project.flows.get(name='My first flow')
    StreamsetsFlow(name='My first flow', description='optional description', ...)

    >>> project.flows.get(name='My second flow')
    DataStageFlow(..., name='My second flow', description='optional description', ...)

.. _preparing_data__flows__editing_a_flow:

Editing a Flow
~~~~~~~~~~~~~~

You can edit a flow in multiple ways.

For starters, you can edit a flow's attributes like ``name`` or ``description``.

.. code-block:: python

    >>> new_flow.description = 'new description for the flow'
    >>> new_flow
    StreamsetsFlow(name='My first flow', description='new description for the flow', ...)

Because streaming flows have many properties related to pipeline, engines, etc., you can edit a streaming flow's configuration through the :py:attr:`Flow.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.configuration>` property. This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object which encapsulates a flow's configuration.
You can print out the configuration and edit it similarly to a :py:class:`dict`.

.. code-block:: python

    >>> new_flow.configuration['retry_pipeline_on_error']
    True
    >>> new_flow.configuration['retry_pipeline_on_error'] = False

Finally, you can edit any flow by editing its stages.
This can include adding a stage, removing a stage, updating a stage's configuration or connecting a stage in a different way than before.
All the operations described are covered in the :ref:`Stage <preparing_data__stages>` documentation.

.. _preparing_data__flows__updating_a_flow:

Updating a Flow
~~~~~~~~~~~~~~~

In the UI, you can update a flow by making changes to the flow and hitting the 'Save' icon to update the flow.

.. image:: ../../_static/images/flows/save_flow_button.png
   :alt: Screenshot of the flow creation page.
   :align: center
   :width: 100%

In the SDK, you can make any changes to a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance
in memory and update it by passing this object to :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method.

This method returns an HTTP response indicating the status of the update operation.

.. code-block:: python

    >>> new_flow.name = 'new flow name'  # you can also update the stages, configuration, etc.
    >>> project.update_flow(new_flow)
    <Response [200]>
    >>> new_flow
    StreamsetsFlow(name='new flow name', description='new description for the flow', ...)

.. _preparing_data__flows__duplicating_a_flow:

Duplicating a Flow
~~~~~~~~~~~~~~~~~~

To duplicate a flow using the SDK, you need to pass a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance
to the :py:meth:`Project.duplicate_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_flow>` method
along with the ``name`` parameter for the name of the new flow and an optional ``description`` parameter.

This will duplicate a flow and return a new instance of :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow`.

.. code-block:: python

    >>> duplicated_flow = project.duplicate_flow(new_flow, name='duplicated flow', description='duplicated flow description')
    >>> duplicated_flow
    StreamsetsFlow(name='duplicated flow', description='duplicated flow description', ...)

.. _preparing_data__flows__deleting_a_flow:

Deleting a Flow
~~~~~~~~~~~~~~~

To delete a flow in the UI, you can go to **Assets**, choose a flow and click on the three dots next to it and choose ``Delete``.

.. image:: ../../_static/images/flows/flow_options.png
   :alt: Screenshot of the flow creation page.
   :align: center
   :width: 100%

To delete a flow via the SDK, you need to pass a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance to the :py:meth:`Project.delete_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_flow>` method.

This method returns an HTTP response indicating the status of the update operation.

.. code-block:: python

    >>> project.delete_flow(duplicated_flow)
    <Response [204]>

.. _preparing_data__flows__validating_a_flow:

Validating a Flow (Streaming)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can validate a streaming flow by making changes to the flow and hitting the 'Validate' icon to validate the flow.

.. image:: ../../_static/images/flows/validate_flow_button.png
   :alt: Screenshot of the validate button.
   :align: center
   :width: 100%

To validate a streaming flow via the SDK, you need to update a flow, and then call the :py:meth:`StreamsetsFlow.validate() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.validate>` method.
This will return a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.ValidationResult` object.
This object contains a ``issues`` attribute that contains a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.FlowValidationError` instances if there are any errors.

.. code-block:: python

    >>> new_flow.add_stage('Trash')
    Trash_01()
    >>> project.update_flow(new_flow)
    <Response [200]>
    >>> new_flow.validate()
    ValidationResult(success=False, issues=[FlowValidationError(type='stageIssues', instanceName='Trash_01', humanReadableMessage='The first stage must be an origin'), FlowValidationError(type='stageIssues', instanceName='Trash_01', humanReadableMessage='Target must have input streams')], message='Validation failed')

.. _preparing_data__flows__previewing_a_flow:



Previewing a flow
~~~~~~~~~~~~~~~~~

In the UI, you can preview a flow by hitting the 'Preview' icon.

.. image:: ../../_static/images/flows/preview_flow_button.png
   :alt: Screenshot of the preview button.
   :align: center
   :width: 100%

To preview a flow via the SDK, you need to call :py:meth:`StreamsetsFlow.preview() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.preview>` method.
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

Exporting Flows
~~~~~~~~~~~~~~~

To export a flow via the SDK, you need to call the :py:meth:`Platform.export_streaming_flows() <ibm_watsonx_data_integration.platform.Platform.export_streaming_flows>` method and
must pass in either an individual :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow` object,
a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlows` object, or a list of
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow` objects. You may also choose to set the
``with_plain_text_credentials`` parameter to export plain text credentials, the ``destination`` parameter to export the flow to a specific location,
and the ``stream`` parameter if they would like to stream the zip file data. The function will return the location the exported zip file was written to.

.. skip: start 'Export Flows API endpoint not on Prod yet'

.. code-block:: python

    >>> project.export_streaming_flows(flows=project.flows)
    PosixPath('flows.zip')

.. skip: end

.. _preparing_data__flows__handling_error_records:

Handling Error Records (Streaming)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To edit error record handling on the UI, click the gear icon on the top-right of the screen in a flow's edit page.

.. image:: ../../_static/images/flows/save_flow_button.png
   :alt: Screenshot of the flow creation page.
   :align: center
   :width: 100%

This opens a new pop-up window with a tab for ``Error records`` on the left. This will let you adjust the error record handling for the flow.

.. image:: ../../_static/images/flows/flow_settings.png
   :alt: Screenshot of the flow creation page.
   :align: center
   :width: 100%

The page lets you change how error records are handled by policy and which stage should handle them.

Let's learn how to change the policy first.

The possible options for error record policy are ``Original record as it was generated by the origin`` and
``Record as it was seen by the stage that sent it to error stream``.
In the SDK, these equate to ``ORIGINAL_RECORD`` and ``STAGE_RECORD``.

This can be updated in a flow's configuration.

.. code-block:: python

    >>> new_flow.configuration['error_record_policy']
    'ORIGINAL_RECORD'
    >>> new_flow.configuration['error_record_policy'] = 'STAGE_RECORD'


To change the error record stage, you can call :py:meth:`StreamsetsFlow.set_error_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.set_error_stage>` method.
You need to pass either the ``label`` or the ``name`` of the new error stage, you can also optionally pass in the new stage's ``library``.

.. note::

    All error stages other than ``Discard`` will have configuration options for you to customize your experience.

.. code-block:: python

    >>> write_to_file = new_flow.set_error_stage('Write to File')
    >>> write_to_file.configuration['directory'] = '/path/to/some/directory'

Finally, you can view the current error stage for a flow at any point using the :py:attr:`StreamsetsFlow.error_stage <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.error_stage>` property.

.. code-block:: python

    >>> new_flow.error_stage
    WritetoFile_ErrorStage()

.. _preparing_data__flows__compiling_a_flow:

Compiling a Flow (Batch)
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can compile a batch flow by hitting the 'Compile' icon to compile the flow. Compiling is required before running a flow, so pressing the 'Run' icon in the UI will automatically compile first.

.. image:: ../../_static/images/flows/compile_batch_flow.png
   :alt: Screenshot of the compile button.
   :align: center
   :width: 100%

Because of this UI behavior, in the SDK, making a job out of a DataStage flow will **automatically compile** the flow for you.
However, if you wish to just compile a flow without creating or running a job for it, you can still call the :py:meth:`DataStageFlow.compile() <ibm_watsonx_data_integration.services.datastage.models.flow.datastage_flow.Datastage.compile>` method.
This will return an HTTP response indicating the status of the compile operation.
