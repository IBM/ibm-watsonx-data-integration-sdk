.. _preparing_data__streaming_stages:

Stages
======

.. include:: ../shared/stages/introduction.rst

The SDK provides many ways to interact with a flow and its stages:
    * Listing all the stages in a flow
    * Adding a stage to a flow
    * Removing a stage from a flow
    * Connecting stages
    * Connecting to Specific Output Streams
    * Listing all the stages connected to a stage
    * Disconnecting stages
    * Chaining stage connect and disconnect operations
    * Editing a stage's configuration

.. _preparing_data__streaming__listing_stages_in_flow:

Listing all Stages in a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all the stages in a flow, you can use the :py:attr:`StreamingFlow.stages <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.stages>` property.
This returns a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> flow.stages
    [DevRawDataSource_01(stage_name='Dev Raw Data Source 1'), Trash_01(stage_name='Trash 1')]


Discovering Available Stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before adding stages to a flow, you can discover what stages are available using the :py:attr:`StreamingFlow.available_stages <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.available_stages>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.common.utils.SeekableList` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageInfo` objects containing information about each available stage.

You can filter the results using the :py:meth:`~ibm_watsonx_data_integration.common.utils.SeekableList.get` or :py:meth:`~ibm_watsonx_data_integration.common.utils.SeekableList.get_all` methods.

.. skip: start 'Python stages might not always be the same'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.streamsets.models.flow_model import StageType
    >>>
    >>> # Get all available stages
    >>> all_stages = flow.available_stages
    >>> len(all_stages)
    245

    >>> # Get a specific stage by label
    >>> dev_gen = flow.available_stages.get(label="Dev Data Generator")
    >>> print(f"{dev_gen.label} - {dev_gen.description}")
    Dev Data Generator - Generates records with the specified field names based on the selected data types

    >>> # Get only origin stages using StageType enum
    >>> origins = flow.available_stages.get_all(stage_type=StageType.SOURCE)
    >>> for stage in origins[:3]:
    ...     print(f"{stage.label} - {stage.description}")
    Dev Data Generator - Generates records with the specified field names based on the selected data types
    Dev Raw Data Source - Generates records based on the configured raw data
    JDBC Query Consumer - Reads database data using a user-provided SQL query

    >>> # Get stages from a specific library
    >>> jdbc_stages = flow.available_stages.get_all(library="streamsets-datacollector-jdbc-lib")

.. skip: end

Each :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageInfo` object contains the following:
    * ``name`` - Internal stage identifier
    * ``label`` - Human-readable display name
    * ``stage_type`` - Stage type as :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageType` enum (SOURCE, TARGET, PROCESSOR, EXECUTOR)
    * ``library`` - Library identifier
    * ``library_label`` - Human-readable library name
    * ``description`` - Stage description
    * ``version`` - Stage version
    * ``config_definitions`` - :py:class:`~ibm_watsonx_data_integration.common.utils.SeekableList` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageConfigDefinition` objects describing configurable properties

.. note::

    The :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageType` enum has four values: ``SOURCE``, ``PROCESSOR``, ``EXECUTOR``, and ``TARGET``.


Discovering Stage Configuration Properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each stage has configurable properties that can be discovered before adding the stage to a flow. The :py:attr:`StreamingStageInfo.config_definitions <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageInfo.config_definitions>` property returns a :py:class:`~ibm_watsonx_data_integration.common.utils.SeekableList` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageConfigDefinition` objects.

Each :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingStageConfigDefinition` contains the following:
    * ``name`` - Configuration property name
    * ``type`` - Configuration property type (e.g., STRING, BOOLEAN, NUMBER)
    * ``required`` - Whether the property is required
    * ``default`` - Default value for the property
    * ``description`` - Description of what the property does
    * ``accepted_values`` - List of accepted values (for enum-type properties)

You can use the :py:meth:`~ibm_watsonx_data_integration.common.utils.SeekableList.get` and :py:meth:`~ibm_watsonx_data_integration.common.utils.SeekableList.get_all` methods to filter configuration properties.

.. note::

    The following examples show illustrative values. Actual configuration properties, their names, types, and default values will vary depending on the stage and its version.

.. skip: start 'Configuration properties may vary'

.. code-block:: python

    >>> # Get a stage's configuration definitions
    >>> dev_gen = flow.available_stages.get(label="Dev Data Generator")
    >>> config_defs = dev_gen.config_definitions
    >>> len(config_defs)
    15

    >>> # Find a specific configuration property
    >>> batch_size_config = config_defs.get(name="batchSize")
    >>> print(f"{batch_size_config.name}: {batch_size_config.description}")
    batchSize: Number of records to be included in each batch

    >>> print(f"Type: {batch_size_config.type}, Required: {batch_size_config.required}")
    Type: NUMBER, Required: False

    >>> print(f"Default: {batch_size_config.default}")
    Default: 1000

    >>> # Get all required configuration properties
    >>> required_configs = config_defs.get_all(required=True)
    >>> for config in required_configs:
    ...     print(f"- {config.name}: {config.description}")
    - dataGenConfigs: Data Generator Configurations

    >>> # Get all optional configuration properties
    >>> optional_configs = config_defs.get_all(required=False)
    >>> print(f"Found {len(optional_configs)} optional properties")
    Found 14 optional properties

.. skip: end

This is particularly useful when you need to understand what properties are available before configuring a stage, or when building dynamic configuration interfaces.


Adding a Stage to a Flow
~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/add_stage_to_flow.rst

.. _preparing_data__add_stage_to_flow_streaming:

In the SDK, you can use the :py:meth:`StreamingFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.add_stage>` method.
This method accepts the following parameters: ``label``, ``name``, ``type``, ``library``, and ``stage_info``.

This method returns an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` representing the newly created stage.

After adding stages, you need to call the :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method to save the changes.

**Method 1: Using label (traditional approach)**

.. code-block:: python

    >>> flow.add_stage(label='Amazon SQS Consumer')
    AmazonSQSConsumer_01(stage_name='Amazon SQS Consumer 1')
    >>> project.update_flow(flow)
    <Response [200]>

**Method 2: Using StreamingStageInfo from available_stages**

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.streamsets.models.flow_model import StageType
    >>>
    >>> # Discover available stages
    >>> origins = flow.available_stages.get_all(stage_type=StageType.SOURCE)
    >>>
    >>> # Find the stage you want using get()
    >>> jdbc_consumer = flow.available_stages.get(label="JDBC Query Consumer")
    >>>
    >>> # Add it using stage_info
    >>> stage = flow.add_stage(stage_info=jdbc_consumer)
    >>> project.update_flow(flow)
    <Response [200]>

You can use the ``type`` parameter to narrow down the type of stage that is returned when multiple stages share the same ``label``.
For example, ``Amazon S3`` can be of ``type`` ``origin``, ``executor`` or ``destination``.
The :py:meth:`StreamingFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.add_stage>` method
returns the first possible stage matching the conditions, therefore it is advisable to narrow down your possibilities by always specifying ``type`` or using ``stage_info``.


.. _preparing_data__streaming__remove_stage_from_flow:

Remove a Stage from a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/remove_stage_from_flow.rst

In the SDK, you can remove a stage from a flow by using the :py:meth:`StreamingFlow.remove_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.remove_stage>` method
and passing an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` to it.

All stages connected to this stage will be disconnected by this action.

This method does not return anything.

After removing stages, you need to call the :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method to save the changes.

.. code-block:: python

    >>> amazon_sqs_consumer = next(filter(lambda stage: stage.instance_name == 'AmazonSQSConsumer_01', flow.stages))
    >>> flow.remove_stage(amazon_sqs_consumer)
    >>> project.update_flow(flow)
    <Response [200]>

.. _preparing_data__streaming__connecting_stages:

Connecting Stages
~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/connecting_stages.rst

In the SDK, to connect streaming stages to each other, you can use the following methods:
    * :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>` - connect the output of the current stage to the input of another stage.
    * :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` - connect the input of the current stage to the output of another stage.
    * :py:meth:`Stage.connect_event_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_event_to>` - connect the event output of the current stage to the input of another stage.

For all the methods listed above, we can pass one or more instances of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` as parameters to connect the stages.

.. code-block:: python

    >>> dev_random_source = flow.add_stage('Dev Raw Data Source')  # a sample origin stage that generates random data
    >>> trash = flow.add_stage('Trash')  # a sample destination stage that accepts all input and discards it
    >>> dev_random_source.connect_output_to(trash)  # alternatively, you can call: trash.connect_input_to(dev_random_source)
    Trash_02(stage_name='Trash 2')
    >>> # events are connected in a similar way
    >>> pipeline_finisher = flow.add_stage('Pipeline Finisher Executor')
    >>> dev_random_source.connect_event_to(pipeline_finisher)  # outputs events to pipeline finisher
    PipelineFinisherExecutor_01(stage_name='Pipeline Finisher Executor 1')
    >>> project.update_flow(flow)
    <Response [200]>


.. _preparing_data__streaming__stage_with_predicates:

Connecting Stages with Multiple Outputs
---------------------------------------

Some stages support multiple outputs determined by ``predicates``. Predicates allow you to route data to different output lanes based on specific conditions.

For stages that support predicates, you can modify the ``predicates`` attribute to control the number of outputs.
This allows you to connect the stage to multiple downstream stages, with each connection routing data based on a specific predicate condition.

The following examples demonstrate how to work with predicates using the ``Stream Selector`` stage, but the same approach applies to any stage that supports predicates.

You can view the predicates of a stage that supports them using the :py:attr:`StageWithPredicates.predicates <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.predicates>` property.

.. code-block:: python

    >>> stream_selector = flow.add_stage('Stream Selector')
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

By default, stages with predicate support have only a single ``default`` predicate. You can add more predicates to suit your needs
by using the :py:meth:`StageWithPredicates.add_predicates() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.add_predicates>` method.

This method accepts a :py:class:`list` of :py:class:`str` containing the predicate expressions you want to add.

.. code-block:: python

    >>> stream_selector.add_predicates(['${record:value("/expense") >= 10000}', '${record:value("/expense") < -10000}'])
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") < -10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") >= 10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

To remove a predicate, pass it to the :py:meth:`StageWithPredicates.remove_predicate() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.remove_predicate>` method.

.. code-block:: python

    >>> stream_selector.remove_predicate(stream_selector.predicates[0])
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") >= 10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

To connect a stage with a specific predicate, use the ``predicate`` parameter in the :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>`
or :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` methods.

.. code-block:: python

    >>> stream_selector.connect_output_to(trash, predicate=stream_selector.predicates[0])
    Trash_02(stage_name='Trash 2')
    >>> # alternatively, you can use:
    >>> trash.connect_input_to(stream_selector, predicate=stream_selector.predicates[0])
    StreamSelector_01(stage_name='Stream Selector 1')

.. important::

    The ``predicates`` attribute is an ordered Python :py:class:`list`.
    The index of a predicate is determined solely by its position in this list (0-based indexing).

    The ``default`` predicate is always the last element in the list.

    The value inside the predicate expression (for example ``${1 == 1}``) does **not** determine its index.

For example:

.. skip: start 'dummy predicate values'

.. code-block:: python

    >>> stream_selector.predicates
        [
            {
                'outputLane': 'StreamSelector_01OutputLane...',
                'predicate': '${1 == 1}'
            },  # index 0
            {
                'outputLane': 'StreamSelector_01OutputLane...',
                'predicate': 'default'
            }   # index 1 (always last)
        ]


In this case:

- ``stream_selector.predicates[0]`` corresponds to the first output lane
- ``stream_selector.predicates[1]`` corresponds to the ``default`` lane

When connecting stages, the predicate index must match the position in the list:

.. code-block:: python

    >>> stream_selector.connect_output_to(trash, predicate=stream_selector.predicates[0])
    >>> stream_selector.connect_output_to(processor, predicate=stream_selector.predicates[1])

.. skip: end

``default`` cannot be used as an index value. It is simply the last element in the ``predicates`` list.


.. _preparing_data__streaming__connecting_to_specific_output_streams:

Connecting to Specific Output Streams
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some stages produce multiple output streams (not to be confused with predicates).
For example, certain processor stages may have multiple output lanes numbered 0, 1, 2, etc.
You can connect to a specific output stream using the ``output_idx`` parameter.

.. note::
    Predicates are used to modify the number of outputs for specific stages like ``Stream Selector``.

    Output streams are the actual outputs that can be connected to other stages.


.. code-block:: python

    >>> # Example: A stage with multiple output streams
    >>> record_deduplicator_stage = flow.add_stage('Record Deduplicator')
    >>> renamer = flow.add_stage('Field Renamer')
    >>> trash_stage = flow.add_stage('Trash')
    >>>
    >>> # Get number of output streams
    >>> len(record_deduplicator_stage.output_lanes)
    2
    >>>
    >>> # Connect to the first output stream (index 0)
    >>> record_deduplicator_stage.connect_output_to(renamer, output_idx=0)
    FieldRenamer_01(stage_name='Field Renamer 1')
    >>>
    >>> # Connect to the second output stream (index 1)
    >>> record_deduplicator_stage.connect_output_to(trash_stage, output_idx=1)
    Trash_03(stage_name='Trash 3')

.. note::
    The ``output_idx`` parameter is zero-based. If not specified, it defaults to 0 (the first output stream).


.. _preparing_data__streaming__listing_connected_stages:

Listing all Stages Connected to a Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three ways a stage can be connected to another stage: it can output data to another stage, it can output event data to another stage,
or it can receive input data from a stage.

There are three properties of a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instance for each of these types of connections:
    * :py:attr:`Stage.inputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.inputs>` - all the stages that input data into the current stage.
    * :py:attr:`Stage.outputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.outputs>` - all the stages that the current stage outputs to.
    * :py:attr:`Stage.events <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.events>` - all the stages that the current stage outputs its events to.

All three properties return a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> dev_random_source.outputs
    [Trash_02(stage_name='Trash 2')]
    >>> dev_random_source.events
    [PipelineFinisherExecutor_01(stage_name='Pipeline Finisher Executor 1')]
    >>> trash.inputs
    [DevRawDataSource_02(stage_name='Dev Raw Data Source 2'), StreamSelector_01(stage_name='Stream Selector 1')]

.. _preparing_data__streaming__disconnecting_stages:

Disconnecting Stages
~~~~~~~~~~~~~~~~~~~~

In the UI, to disconnect a stage, you can click on the connection and then the delete icon that comes above it.

.. image:: /_static/images/stages/disconnect_stages.png
   :alt: Screenshot for disconnecting a stage.
   :align: center
   :width: 85%

To disconnect stages, we have a similar trio of methods as for connecting:
    * :py:meth:`Stage.disconnect_output_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_output_from>` - disconnect the output of the current stage from the input of another stage.
    * :py:meth:`Stage.disconnect_input_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_input_from>` - disconnect the input of the current stage from the output of another stage.
    * :py:meth:`Stage.disconnect_event_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_event_from>` - disconnect the event output of the current stage from the input of another stage.

.. code-block:: python

    >>> dev_random_source.disconnect_output_from(trash)  # alternatively, you can call: trash.disconnect_input_from(dev_random_source)
    Trash_02(stage_name='Trash 2')
    >>> dev_random_source.disconnect_event_from(pipeline_finisher)
    PipelineFinisherExecutor_01(stage_name='Pipeline Finisher Executor 1')


.. _preparing_data__streaming__chaining_stage_connect_disconnect:

Chaining Stage's Connect and Disconnect Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of connecting each stage one by one, it is possible to write everything in a single line of code by using chaining:

.. code-block:: python

    >>> source = flow.add_stage('Dev Data Generator')
    >>> processor = flow.add_stage('Field Remover')
    >>> trash = flow.add_stage('Trash')
    >>> source.connect_output_to(processor).connect_output_to(trash)
    Trash_04(stage_name='Trash 4')

It is also possible to connect multiple stages to the output of a stage with a single call.

.. code-block:: python

    >>> source = flow.add_stage('Dev Data Generator')
    >>> renamer1 = flow.add_stage('Field Renamer')
    >>> renamer2 = flow.add_stage('Field Renamer')
    >>> source.connect_output_to(renamer1, renamer2)
    [FieldRenamer_02(stage_name='Field Renamer 2'), FieldRenamer_03(stage_name='Field Renamer 3')]


.. _preparing_data__streaming__editing_stage_configuration:

Editing a Stage's Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stages have properties that can be configured.
You can edit a stage's configuration through the :py:attr:`Stage.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.configuration>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object that encapsulates a stage's configuration.
You can print the configuration and edit it similarly to a :py:class:`dict`.

.. code-block:: python

    >>> dev_random_source.configuration['stop_after_first_batch']
    False
    >>> dev_random_source.configuration['stop_after_first_batch'] = True
