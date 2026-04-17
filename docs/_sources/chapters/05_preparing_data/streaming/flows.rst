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

To create a flow using the SDK, make sure you have completed the following steps:
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
    Creating a streaming flow in the UI requires setting an environment.

.. image:: /_static/images/flows/create_streaming_flow.png
    :alt: Screenshot of the flow creation page.
    :align: center
    :width: 100%

In the SDK, you can create a flow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_flow>` method.
You are required to supply the ``name`` and ``environment`` parameters.
To learn how to create an environment, see :ref:`creating an environment <projects__environments__creating_an_environment>`.
You can also provide an optional ``description``.
You do not need to specify ``flow_type`` because its default value is ``streaming``.
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
If you want to retrieve only streaming flows, use the :py:meth:`Project.flows.get_all() <ibm_watsonx_data_integration.common.models.CollectionModel.get_all>` method to filter by ``flow_type``.
You can also retrieve a single flow using the :py:meth:`Project.flows.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method, which takes unique identifiers such as ``flow_id`` or ``name``.

.. code-block:: python

    >>> project.flows  # a list of all flows
    [StreamingFlow(...), StreamingFlow(...)...]
    >>> project.flows.get_all(flow_type='streaming')
    [..., StreamingFlow(name='My streaming flow', description='optional description', ...)...]
    >>> project.flows.get(name='My streaming flow')
    StreamingFlow(name='My streaming flow', description='optional description', ...)

.. _preparing_data__streaming__flows__editing_a_flow:

Editing a Flow
~~~~~~~~~~~~~~

.. include:: ../shared/flows/editing_flow.rst

In addition, streaming flows have many properties related to pipelines, engines, and more. You can edit a streaming flow's configuration through the :py:attr:`Flow.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.configuration>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object that encapsulates a flow's configuration.
You can print the configuration and edit it similarly to a :py:class:`dict`.

.. code-block:: python

    >>> new_flow.configuration['retry_pipeline_on_error']
    True
    >>> new_flow.configuration['retry_pipeline_on_error'] = False


.. _preparing_data__streaming__flows__arranging_stages:

Arranging Stages
~~~~~~~~~~~~~~~~

When you add stages to a streaming flow programmatically, they are positioned in a simple left-to-right layout by default.
For complex flows with multiple branches and connections, you may want to automatically arrange the stages for better visualization.

In the UI, there is an 'Auto Arrange' button that organizes stages based on their connections.
In the SDK, you can achieve a similar result by calling the :py:meth:`StreamingFlow.auto_arrange() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.auto_arrange>` method.
This method positions stages based on their input and output lanes.

.. code-block:: python

    >>> new_flow.auto_arrange()
    StreamingFlow(name='My streaming flow', ...)

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

In the UI, you can validate a streaming flow by making changes to the flow and clicking the **Validate** icon.

.. image:: /_static/images/flows/validate_flow_button.png
   :alt: Screenshot of the validate button.
   :align: center
   :width: 100%

To validate a streaming flow via the SDK, first update the flow and then call the :py:meth:`StreamingFlow.validate() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.validate>` method.
This returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.ValidationResult` object.
This object contains an ``issues`` attribute with a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.FlowValidationError` instances if there are any errors.

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

In the UI, you can preview a flow by clicking the **Preview** icon.

.. image:: /_static/images/flows/preview_flow_button.png
   :alt: Screenshot of the preview button.
   :align: center
   :width: 100%

To preview a flow via the SDK, call the :py:meth:`StreamingFlow.preview() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.preview>` method.
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

To import a flow via the SDK, call the :py:meth:`Project.import_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.import_flows>` method and
specify the ``source`` parameter, which is the path to the ZIP file containing the JSON file or files for the streaming flow or flows to be imported, along with the ``type`` parameter, which should be
``'streaming'``.

You must also set the ``conflict_resolution`` parameter, which determines how to handle an attempted import of a duplicate flow that already exists in the project. The options for ``conflict_resolution`` are listed below:
    * ``'skip'`` – skip this particular flow and move to the next flow to be imported.
    * ``'replace'`` - replace the existing flow with the flow to be imported.
    * ``'rename'`` - keep the existing flow and rename the flow that is being imported.

The function returns either an imported :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow`
or a list of imported :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` objects.

.. skip: start 'Import Flows API endpoint not on Prod yet'

.. code-block:: python

    >>> project.import_flows(type='streaming', source='flows_to_import.zip', conflict_resolution='skip')
    StreamingFlow(name='dummy_flow', description='dummy_description', flow_id='...', engine_version='...')

.. skip: end

.. _preparing_data__streaming__flows__handling_error_records:

Handling Error Records
~~~~~~~~~~~~~~~~~~~~~~

To edit error record handling in the UI, click the gear icon in the top-right corner of the screen on a flow's edit page.

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
You need to pass either the ``label`` or the ``name`` of the new error stage. You can also optionally pass the new stage's ``library``.

.. note::

    All error stages other than ``Discard`` will have configuration options for you to customize your experience.

.. code-block:: python

    >>> write_to_file = new_flow.set_error_stage('Write to File')
    >>> write_to_file.configuration['directory'] = '/path/to/some/directory'

You can view the current error stage for a flow at any point using the :py:attr:`StreamingFlow.error_stage <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.error_stage>` property.

.. code-block:: python

    >>> new_flow.error_stage
    WritetoFile_ErrorStage(stage_name='Error Records - Write to File')

.. _preparing_data__streaming__flows__using_parameter_sets:

Using Parameter Sets
~~~~~~~~~~~~~~~~~~~~

Parameter sets allow you to make your streaming flows more flexible by defining reusable parameters that can be referenced throughout your flow.

In the UI, you can use a Parameter Set in a streaming flow by navigating to **Flow parameters -> Parameter sets -> Add parameter sets** and choosing the desired parameter set from the list.

In the SDK, to use a Parameter Set instance in a streaming flow, you can pass it to the :py:meth:`StreamingFlow.use_parameter_set() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.use_parameter_set>` method.
This method establishes a relationship between the flow and the parameter set, and automatically adds the parameter set's parameters as constants in the flow's configuration.

.. code-block:: python

    >>> # Retrieve the parameter set
    >>> paramset = project.parameter_sets.get(name='my_streaming_params')
    >>>
    >>> # Use the parameter set in the streaming flow
    >>> streaming_flow_with_parameters.use_parameter_set(paramset)
    StreamingFlow(name='flow with parameters', ...)
    >>>
    >>> # Update the flow to save the changes
    >>> project.update_flow(streaming_flow_with_parameters)
    <Response [200]>

To use a parameter from a parameter set in your streaming flow stages, reference it using the notation: ``${parameter_set_name__parameter_name}`` (note the double underscore ``__`` separator).

.. code-block:: python

    >>> # Example: Using parameters in stage configurations
    >>> dev_raw_data = streaming_flow_with_parameters.add_stage('Dev Raw Data Source')
    >>> dev_raw_data.data_format = '${my_streaming_params__data_format}'
    >>> dev_raw_data.number_of_threads = '${my_streaming_params__number_of_threads}'
    >>> dev_raw_data.stop_after_first_batch = '${my_streaming_params__stop_after_first_batch}'

You can retrieve all parameter sets associated with a streaming flow using the :py:attr:`StreamingFlow.parameter_sets <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.parameter_sets>` property.

.. code-block:: python

    >>> streaming_flow_with_parameters.parameter_sets
    [ParameterSet(name='my_streaming_params', parameters=[...], description='', value_sets=[])]

.. note::
    Streaming flows currently support only :py:class:`ParameterType.String <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterType>` type parameters.

    Streaming flows do not support:

    - Local parameters
    - PROJDEF parameter sets
    - Value sets for parameter sets

    For more information about parameter sets, see :ref:`Parameter Sets <preparing_data__parameter_sets>`.
