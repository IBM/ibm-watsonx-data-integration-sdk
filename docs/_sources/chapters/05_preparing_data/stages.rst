.. _preparing_data__stages:

Stages
======
|

A stage is a part of a flow, a stage is responsible for processing the flow's data in a single simple way.
This way we can connect a bunch of stages to create a flow that can do complex tasks.

The SDK provides many ways to interact between a flow and its stages:
    * Listing all the stages in a flow
    * Adding a stage to a flow
    * Removing a stage from a flow
    * Connecting stages
    * Listing all the stages connected to a stage
    * Disconnecting stages
    * Editing a stage's configuration

.. _preparing_data__listing_stages_in_flow:

Listing all Stages in a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all the stages in a flow, you can use the :py:attr:`StreamsetsFlow.stages <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.stages>` property.
This returns a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> flow.stages
    [DevRawDataSource_01(), Trash_01()]

.. warning::
   This method currently only applies to StreamSets flows.

.. _preparing_data__add_stage_to_flow:

Add a Stage to a Flow
~~~~~~~~~~~~~~~~~~~~~~

To add stages to a flow in the UI, you can open the dropdown menu on the left of the flow edit page and select the stage you want to add.

.. image:: ../../_static/images/stages/add_stage.png
   :alt: Screenshot for adding a stage.
   :align: center
   :width: 40%

.. _preparing_data__add_stage_to_flow_streaming:

Streaming
---------

In the SDK, you can use the :py:meth:`StreamsetsFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.add_stage>` method.
This method accepts the following parameters ``label``, ``name``, ``type`` and ``library``, of which one of ``label`` or ``name`` must be provided.

This method returns an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` representing the newly created stage.

.. code-block:: python

    >>> amazon_sqs_consumer = flow.add_stage(label='Amazon SQS Consumer')

You can use the ``type`` parameter to narrow down on the type of stage that is returned when multiple stages share the same ``label``.
For example, ``Amazon S3`` can be of ``type`` ``origin``, ``executor`` or ``destination``.
The :py:meth:`StreamsetsFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.add_stage>` method
returns the first possible stage matching the conditions, therefore it is advisable to narrow down your possibilities by always specifying ``type``.

.. note::

    There are four possible values for ``type`` namely, ``origin``, ``processor``, ``executor`` and ``destination``.

.. _preparing_data__add_stage_to_flow_batch:

Batch
-----

In the SDK, you can use the :py:meth:`DataStageFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.datastage_flow.DataStageFlow.add_stage>` method.
This method takes in the stage's official ``type`` and the ``label`` for that stage. The ``type`` is the unique identifier for that stage and the ``label`` is the name you wish to give it.

This method returns the newly created stage.

.. code-block:: python

    >>> amazon_rds = batch_flow.add_stage(type = 'Amazon RDS for PostgreSQL', label = 'Amazon RDS')
    >>> project.update_flow(batch_flow)
    <Response [201]>

You can find a list of all unique identifiers here: :py:meth:`DataStageFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.datastage_flow.DataStageFlow.add_stage>`

.. _preparing_data__remove_stage_from_flow:

Remove a Stage from a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~

To remove a stage in the UI, you can click on the stage and then click on the delete icon that comes above it.

.. image:: ../../_static/images/stages/remove_stage.png
   :alt: Screenshot for removing a stage.
   :align: center
   :width: 50%

In the SDK, you can remove a stage from a flow using the :py:meth:`StreamsetsFlow.remove_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow.remove_stage>` method
and passing an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` to it.

All stages connected to this stage will be disconnected by this action.

This method does not return anything.

.. code-block:: python

    >>> flow.remove_stage(amazon_sqs_consumer)

.. warning::
   This method currently only applies to StreamSets flows.

.. _preparing_data__connecting_stages:

Connecting Stages
~~~~~~~~~~~~~~~~~

In the UI, to connect stages, you can click on the output of a stage and drag it to another stage.

.. image:: ../../_static/images/stages/connect_stages.png
   :alt: Screenshot for connecting a stage.
   :align: center
   :width: 85%

.. _preparing_data__connecting_stages_streaming:

Streaming
---------

In the SDK, to connect streaming stages to each other we can use the following methods:
    * :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>` - this method is used to connect the output of the current stage to the input of another stage.
    * :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` - this method is used to connect the input of the current stage to the output of another stage.
    * :py:meth:`Stage.connect_event_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_event_to>` - this method is used to connect the event output of the current stage to the input of another stage.

For all the methods listed above, we can pass one or more instances of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` as parameters to connect the stages.

.. code-block:: python

    >>> dev_random_source = flow.add_stage('Dev Raw Data Source')  # a sample origin stage that generates random data
    >>> trash = flow.add_stage('Trash')  # a sample destination stage that accepts all input and discards it
    >>> dev_random_source.connect_output_to(trash)  # alternatively, you can call: trash.connect_input_to(dev_random_source)
    >>> # events are connected in a similar way
    >>> pipeline_finisher = flow.add_stage('Pipeline Finisher Executor')
    >>> dev_random_source.connect_event_to(pipeline_finisher)  # outputs events to pipeline finisher

.. _preparing_data__stage_with_predicates:

Connecting Streaming Stages with Multiple Outputs
-------------------------------------------------

There is a special case of ``Stream Selector`` - a stage having multiple outputs. The number of outputs of this stage are determined by ``predicates``.

It is possible to modify the ``predicates`` attribute of a ``Stream Selector`` stage, this causes the number of outputs of the stage to differ.
It is then possible to connect the stage to multiple stages, and we can connect each stage to take the output for a specific predicate.

To do this via the SDK, we will first edit the ``predicates`` of a ``Stream Selector`` stage and then how to connect other stages based on a predicate.

You can view the predicates of a ``Stream Selector`` using the :py:attr:`StageWithPredicates.predicates <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.predicates>` property.

.. code-block:: python

    >>> stream_selector = flow.add_stage('Stream Selector')
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

A ``Stream Selector`` stage has only a single ``default`` predicate by default. We need to add more predicates to suit our needs.
We can do this via the :py:meth:`StageWithPredicates.add_predicates() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.add_predicates>` method.

We need to pass a :py:class:`list` of :py:class:`str` which contains the predicates we want to add.

.. code-block:: python

    >>> stream_selector.add_predicates(['${record:value("/expense") >= 10000}', '${record:value("/expense") < -10000}'])
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") < -10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") >= 10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

To remove a predicate we need to pass a predicate into the :py:meth:`StageWithPredicates.remove_predicate() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StageWithPredicates.remove_predicate>` method.

.. code-block:: python

    >>> stream_selector.remove_predicate(stream_selector.predicates[0])
    >>> stream_selector.predicates
    [{'outputLane': 'StreamSelector_01OutputLane...', 'predicate': '${record:value("/expense") >= 10000}'}, {'outputLane': 'StreamSelector_01OutputLane...', 'predicate': 'default'}]

Finally, to connect a stage with a specific predicate, use the ``predicate`` parameter in the :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>`
or :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` methods.

.. code-block:: python

    >>> stream_selector.connect_output_to(trash, predicate=stream_selector.predicates[0])
    >>> # alternatively, you can use:
    >>> trash.connect_input_to(stream_selector, predicate=stream_selector.predicates[0])

.. _preparing_data__connecting_stages_batch:

Batch
-----

In the SDK, to connect batch stages to each other we can use the :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode.connect_output_to>` method.
This method connects the stage it's called on to the stage in the argument and returns the created link between the two stages.

.. code-block:: python

    >>> row_gen = batch_flow.add_stage('Row Generator', 'Row_Generator') # a sample origin stage that generates data
    >>> peek = batch_flow.add_stage('Peek', 'Peek') # a sample destination stage that outputs all input to console
    >>> link = row_gen.connect_output_to(peek)

.. _preparing_data__listing_connected_stages:

Listing all Stages Connected to a Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three ways a stage can be connected to another stage, it can output data to another stage, it can output event data to another stage
or it could get input data from a stage.

There are three properties of a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instance for each of these types of connections:
    * :py:attr:`Stage.inputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.inputs>` - for all the stages that input data into the current stage.
    * :py:attr:`Stage.outputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.outputs>` - for all the stages that the current stage outputs to.
    * :py:attr:`Stage.events <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.events>` - for all the stages that the current stage outputs its events to.

All three properties return a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> dev_random_source.outputs
    [Trash_02()]
    >>> dev_random_source.events
    [PipelineFinisherExecutor_01()]
    >>> trash.inputs
    [DevRawDataSource_02(), StreamSelector_01()]

.. warning::
   This method currently only applies to StreamSets flows.

.. _preparing_data__disconnecting_stages:

Disconnecting Stages
~~~~~~~~~~~~~~~~~~~~

In the UI, to disconnect a stage, you can click on the connection and then the delete icon that comes above it.

.. image:: ../../_static/images/stages/disconnect_stages.png
   :alt: Screenshot for disconnecting a stage.
   :align: center
   :width: 85%

To disconnect stages, we have a similar trio of methods as for connecting:
    * :py:meth:`Stage.disconnect_output_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_output_from>` - this method is used to disconnect the output of the current stage from the input of another stage.
    * :py:meth:`Stage.disconnect_input_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_input_from>` - this method is used to disconnect the input of the current stage from the output of another stage.
    * :py:meth:`Stage.disconnect_event_from() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.disconnect_event_from>` - this method is used to disconnect the event output of the current stage from the input of another stage.

.. code-block:: python

    >>> dev_random_source.disconnect_output_from(trash)  # alternatively, you can call: trash.disconnect_input_from(dev_random_source)
    >>> dev_random_source.disconnect_event_from(pipeline_finisher)

.. warning::
   This method currently only applies to StreamSets flows.

.. _preparing_data__editing_stage_configuration:

Editing a Stage's Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stages have properties which can be configured.

.. _preparing_data__editing_stage_configuration_streaming:

Streaming
---------

You can edit a stage's configuration through the :py:attr:`Stage.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.configuration>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object which encapsulates a stage's configuration.
You can print out the configuration and edit it similar to a :py:class:`dict`.

.. code-block:: python

    >>> dev_random_source.configuration['stop_after_first_batch']
    False
    >>> dev_random_source.configuration['stop_after_first_batch'] = True

.. _preparing_data__editing_stage_configuration_batch:

Batch
-----
You can edit a stage's configuration through its corresponding `configuration` property.
This property returns an object containing all of that stage's properties. The object has a class corresponding to the stage's class.

.. Furthermore, the types of the properties depend on the types in the UI. For example, integer parameters will be integers where possible, checkboxes will be booleans where possible, and choices like dropdowns turn into enums.

.. code-block:: python

    >>> row_gen.configuration
    row_generator(buf_mode=<BufMode.default: 'default'>, max_mem_buf_size=3145728, ...)
    >>> row_gen.configuration.records = 5
    >>> row_gen.configuration.runtime_column_propagation = False
