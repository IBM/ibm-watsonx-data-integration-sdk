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

.. _preparing_data__streaming__flows__supported_flow_architectures:

Supported Flow Architectures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Streaming flows support several common connection patterns. You can combine these patterns within the same flow depending on your processing requirements:

Linear Flows
^^^^^^^^^^^^
One stage connects to one downstream stage in sequence:

.. code-block:: python

    >>> linear_flow = project.create_flow(name='Linear flow', description='optional description', environment=environment)
    >>> source = linear_flow.add_stage('Dev Raw Data Source')
    >>> processor = linear_flow.add_stage('Field Renamer')
    >>> destination = linear_flow.add_stage('Trash')
    >>>
    >>> # Linear connection: source → processor → destination
    >>> source.connect_output_to(processor)
    FieldRenamer_01(name='Field Renamer 1')
    >>> processor.connect_output_to(destination)
    Trash_01(name='Trash 1')

This creates a topology like:

.. code-block:: text

    Source ─→ Processor ─→ Destination

Fan-out Flows
^^^^^^^^^^^^^
One stage connects to multiple downstream stages:

.. code-block:: python

    >>> fan_out_flow = project.create_flow(name='Fan-out flow', description='optional description', environment=environment)
    >>> source = fan_out_flow.add_stage('Dev Raw Data Source')
    >>> dest1 = fan_out_flow.add_stage('Trash')
    >>> dest2 = fan_out_flow.add_stage('Trash')
    >>> dest3 = fan_out_flow.add_stage('Trash')
    >>>
    >>> # Fan-out: source connects to multiple destinations
    >>> source.connect_output_to(dest1, dest2, dest3)
    [Trash_01(name='Trash 1'), Trash_02(name='Trash 2'), Trash_03(name='Trash 3')]

This creates a topology like:

.. code-block:: text

    Source
      ├─→ Destination 1
      ├─→ Destination 2
      └─→ Destination 3

Conditional Routing Flows
^^^^^^^^^^^^^^^^^^^^^^^^^
Stages with multiple outputs can route records to different downstream stages based on predicates:

.. code-block:: python

    >>> conditional_flow = project.create_flow(name='Conditional Routing Flow', description='optional description', environment=environment)
    >>> source = conditional_flow.add_stage('Dev Raw Data Source')
    >>> stream_selector = conditional_flow.add_stage('Stream Selector')
    >>> high_value = conditional_flow.add_stage('Trash')
    >>> low_value = conditional_flow.add_stage('Trash')
    >>> default_dest = conditional_flow.add_stage('Trash')
    >>>
    >>> # Add predicates for conditional routing
    >>> stream_selector.add_predicates([
    ...     '${record:value(\'/amount\') > 1000}',
    ...     '${record:value(\'/amount\') < 100}'
    ... ])
    >>>
    >>> # Connect source to stream selector
    >>> source.connect_output_to(stream_selector)
    StreamSelector_01(name='Stream Selector 1')
    >>>
    >>> # Route to different destinations based on predicates
    >>> stream_selector.connect_output_to(high_value, predicate=stream_selector.predicates[0])
    Trash_01(name='Trash 1')
    >>> stream_selector.connect_output_to(low_value, predicate=stream_selector.predicates[1])
    Trash_02(name='Trash 2')
    >>> stream_selector.connect_output_to(default_dest, predicate=stream_selector.predicates[2])
    Trash_03(name='Trash 3')

This creates a topology like:

.. code-block:: text

    Source
      │
      ▼
    Stream Selector
      ├─→ High Value Destination (amount > 1000)
      ├─→ Low Value Destination (amount < 100)
      └─→ Default Destination (all others)

Converging Flows (Fan-In)
^^^^^^^^^^^^^^^^^^^^^^^^^
Multiple upstream stages connect into a downstream stage:

.. code-block:: python

    >>> converging_flow = project.create_flow(name='Converging Flow', description='optional description', environment=environment)
    >>> source1 = converging_flow.add_stage('Dev Raw Data Source')
    >>> source2 = converging_flow.add_stage('Dev Raw Data Source')
    >>> source3 = converging_flow.add_stage('Dev Raw Data Source')
    >>> destination = converging_flow.add_stage('Trash')
    >>>
    >>> # Converging: multiple sources connect to one destination
    >>> source1.connect_output_to(destination)
    Trash_01(name='Trash 1')
    >>> source2.connect_output_to(destination)
    Trash_01(name='Trash 1')
    >>> source3.connect_output_to(destination)
    Trash_01(name='Trash 1')

This creates a topology like:

.. code-block:: text

    Source 1 ─┐
              │
    Source 2 ─┼─→ Destination
              │
    Source 3 ─┘

Diamond Flows
^^^^^^^^^^^^^
Data splits into multiple branches that later converge back into a single stage, allowing parallel processing paths.

.. code-block:: python

    >>> diamond_flow = project.create_flow(name='Diamond Flow', description='optional description', environment=environment)
    >>> source = diamond_flow.add_stage('Dev Raw Data Source')
    >>>
    >>> # Split into two branches
    >>> branch1_processor = diamond_flow.add_stage('Field Renamer')
    >>> branch2_processor = diamond_flow.add_stage('Field Masker')
    >>>
    >>> # Converge back to single destination
    >>> destination = diamond_flow.add_stage('Trash')
    >>>
    >>> # Create diamond topology
    >>> source.connect_output_to(branch1_processor, branch2_processor)
    [FieldRenamer_01(name='Field Renamer 1'), FieldMasker_01(name='Field Masker 1')]
    >>> branch1_processor.connect_output_to(destination)
    Trash_01(name='Trash 1')
    >>> branch2_processor.connect_output_to(destination)
    Trash_01(name='Trash 1')

This creates a topology like:

.. code-block:: text

    Source
      ├─→ Branch 1 Processor ─┐
      │                       ├─→ Destination
      └─→ Branch 2 Processor ─┘

Multistage Branching Flows
^^^^^^^^^^^^^^^^^^^^^^^^^^
Complex flows with multiple levels of branching and processing, where different branches can have their own processing pipelines.

.. code-block:: python

    >>> multistage_flow = project.create_flow(name='Multistage Flow', description='optional description', environment=environment)
    >>> source = multistage_flow.add_stage('Dev Raw Data Source')
    >>>
    >>> # First level branching
    >>> stream_selector = multistage_flow.add_stage('Stream Selector')
    >>> stream_selector.add_predicates(['${record:value(\'/type\') == \'A\'}', '${record:value(\'/type\') == \'B\'}'])
    >>>
    >>> # Branch A: Multiple processing stages
    >>> branch_a_processor1 = multistage_flow.add_stage('Field Renamer')
    >>> branch_a_processor2 = multistage_flow.add_stage('Field Masker')
    >>> branch_a_dest = multistage_flow.add_stage('Trash')
    >>>
    >>> # Branch B: Different processing pipeline
    >>> branch_b_processor1 = multistage_flow.add_stage('Expression Evaluator')
    >>> branch_b_processor2 = multistage_flow.add_stage('Field Remover')
    >>> branch_b_dest = multistage_flow.add_stage('Trash')
    >>>
    >>> # Connect source to stream selector
    >>> source.connect_output_to(stream_selector)
    StreamSelector_01(name='Stream Selector 1')
    >>>
    >>> # Connect Branch A (type == 'A')
    >>> stream_selector.connect_output_to(branch_a_processor1, predicate=stream_selector.predicates[0])
    FieldRenamer_01(name='Field Renamer 1')
    >>> branch_a_processor1.connect_output_to(branch_a_processor2)
    FieldMasker_01(name='Field Masker 1')
    >>> branch_a_processor2.connect_output_to(branch_a_dest)
    Trash_01(name='Trash 1')
    >>>
    >>> # Connect Branch B (default)
    >>> stream_selector.connect_output_to(branch_b_processor1, predicate=stream_selector.predicates[1])
    ExpressionEvaluator_01(name='Expression Evaluator 1')
    >>> branch_b_processor1.connect_output_to(branch_b_processor2)
    FieldRemover_01(name='Field Remover 1')
    >>> branch_b_processor2.connect_output_to(branch_b_dest)
    Trash_02(name='Trash 2')

This creates a topology like:

.. code-block:: text

    Source
      │
      ▼
    Stream Selector
      ├─→ Branch A Processor 1 ─→ Branch A Processor 2 ─→ Branch A Destination
      │
      └─→ Branch B Processor 1 ─→ Branch B Processor 2 ─→ Branch B Destination

Event-Driven Side Flows
^^^^^^^^^^^^^^^^^^^^^^^
A stage's event output can be connected separately from its data output:

.. code-block:: python

    >>> event_flow = project.create_flow(name='Event Flow', description='optional description', environment=environment)
    >>> source = event_flow.add_stage('Dev Raw Data Source')
    >>> destination = event_flow.add_stage('Trash')
    >>> executor = event_flow.add_stage('Pipeline Finisher Executor')
    >>>
    >>> # Connect data output
    >>> source.connect_output_to(destination)
    Trash_01(name='Trash 1')
    >>>
    >>> # Connect event output separately
    >>> source.connect_event_to(executor)
    PipelineFinisherExecutor_01(name='Pipeline Finisher Executor 1')

This creates a topology like:

.. code-block:: text

    Source
      ├─→ Destination (data output)
      └─→ Executor (event output)

These architectures are built using the same stage connection APIs described in :ref:`preparing_data__streaming_stages`. For complex topologies with multiple branches, you can also use :py:meth:`StreamingFlow.auto_arrange() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.auto_arrange>` to improve the visual layout after making changes.

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
    Trash_01(name='Trash 1')
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
    WritetoFile_ErrorStage(name='Error Records - Write to File')

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
