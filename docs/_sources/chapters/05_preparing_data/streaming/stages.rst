.. _preparing_data__stages:

Stages
======
|

.. include:: ../shared/stages/introduction.rst

The SDK provides many ways to interact between a flow and its stages:
    * Listing all the stages in a flow
    * Adding a stage to a flow
    * Removing a stage from a flow
    * Connecting stages
    * Listing all the stages connected to a stage
    * Disconnecting stages
    * Chaining Stage's connect and disconnect
    * Editing a stage's configuration

.. _preparing_data__streaming__listing_stages_in_flow:

Listing all Stages in a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all the stages in a flow, you can use the :py:attr:`StreamingFlow.stages <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.stages>` property.
This returns a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> flow.stages
    [DevRawDataSource_01(stage_id='DevRawDataSource_01'), Trash_01(stage_id='Trash_01')]


Adding a Stage to a Flow
~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/add_stage_to_flow.rst

.. _preparing_data__add_stage_to_flow_streaming:

In the SDK, you can use the :py:meth:`StreamingFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.add_stage>` method.
This method accepts the following parameters ``label``, ``name``, ``type`` and ``library``, of which one of ``label`` or ``name`` must be provided.

This method returns an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` representing the newly created stage.

After adding stages you need to call :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method to save the changes.

.. code-block:: python

    >>> flow.add_stage(label='Amazon SQS Consumer')
    AmazonSQSConsumer_01(stage_id='AmazonSQSConsumer_01')
    >>> project.update_flow(flow)
    <Response [200]>

You can use the ``type`` parameter to narrow down on the type of stage that is returned when multiple stages share the same ``label``.
For example, ``Amazon S3`` can be of ``type`` ``origin``, ``executor`` or ``destination``.
The :py:meth:`StreamingFlow.add_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.add_stage>` method
returns the first possible stage matching the conditions, therefore it is advisable to narrow down your possibilities by always specifying ``type``.

.. note::

    There are four possible values for ``type`` namely, ``origin``, ``processor``, ``executor`` and ``destination``.


.. _preparing_data__streaming__remove_stage_from_flow:

Remove a Stage from a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/remove_stage_from_flow.rst

In the SDK, you can remove a stage from a flow using the :py:meth:`StreamingFlow.remove_stage() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow.remove_stage>` method
and passing an instance of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` to it.

All stages connected to this stage will be disconnected by this action.

This method does not return anything.

After removing stages you need to call :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method to save the changes.

.. code-block:: python

    >>> amazon_sqs_consumer = next(filter(lambda stage: stage.instance_name == 'AmazonSQSConsumer_01', flow.stages))
    >>> flow.remove_stage(amazon_sqs_consumer)
    >>> project.update_flow(flow)
    <Response [200]>

.. _preparing_data__streaming__connecting_stages:

Connecting Stages
~~~~~~~~~~~~~~~~~

.. include:: ../shared/stages/connecting_stages.rst

In the SDK, to connect streaming stages to each other we can use the following methods:
    * :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>` - connect the output of the current stage to the input of another stage.
    * :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` - connect the input of the current stage to the output of another stage.
    * :py:meth:`Stage.connect_event_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_event_to>` - connect the event output of the current stage to the input of another stage.

For all the methods listed above, we can pass one or more instances of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` as parameters to connect the stages.

.. code-block:: python

    >>> dev_random_source = flow.add_stage('Dev Raw Data Source')  # a sample origin stage that generates random data
    >>> trash = flow.add_stage('Trash')  # a sample destination stage that accepts all input and discards it
    >>> dev_random_source.connect_output_to(trash)  # alternatively, you can call: trash.connect_input_to(dev_random_source)
    Trash_02(stage_id='Trash_02')
    >>> # events are connected in a similar way
    >>> pipeline_finisher = flow.add_stage('Pipeline Finisher Executor')
    >>> dev_random_source.connect_event_to(pipeline_finisher)  # outputs events to pipeline finisher
    PipelineFinisherExecutor_01(stage_id='PipelineFinisherExecutor_01')
    >>> project.update_flow(flow)
    <Response [200]>


.. _preparing_data__streaming__stage_with_predicates:

Connecting Stages with Multiple Outputs
---------------------------------------

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

To connect a stage with a specific predicate, use the ``predicate`` parameter in the :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_output_to>`
or :py:meth:`Stage.connect_input_to() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.connect_input_to>` methods.

.. code-block:: python

    >>> stream_selector.connect_output_to(trash, predicate=stream_selector.predicates[0])
    Trash_02(stage_id='Trash_02')
    >>> # alternatively, you can use:
    >>> trash.connect_input_to(stream_selector, predicate=stream_selector.predicates[0])
    StreamSelector_01(stage_id='StreamSelector_01')

.. _preparing_data__streaming__listing_connected_stages:

Listing all Stages Connected to a Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three ways a stage can be connected to another stage, it can output data to another stage, it can output event data to another stage
or it can get input data from a stage.

There are three properties of a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instance for each of these types of connections:
    * :py:attr:`Stage.inputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.inputs>` - all the stages that input data into the current stage.
    * :py:attr:`Stage.outputs <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.outputs>` - all the stages that the current stage outputs to.
    * :py:attr:`Stage.events <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.events>` - all the stages that the current stage outputs its events to.

All three properties return a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage` instances.

.. code-block:: python

    >>> dev_random_source.outputs
    [Trash_02(stage_id='Trash_02')]
    >>> dev_random_source.events
    [PipelineFinisherExecutor_01(stage_id='PipelineFinisherExecutor_01')]
    >>> trash.inputs
    [DevRawDataSource_02(stage_id='DevRawDataSource_02'), StreamSelector_01(stage_id='StreamSelector_01')]

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
    Trash_02(stage_id='Trash_02')
    >>> dev_random_source.disconnect_event_from(pipeline_finisher)
    PipelineFinisherExecutor_01(stage_id='PipelineFinisherExecutor_01')


.. _preparing_data__streaming__chaining_stage_connect_disconnect:

Chaining Stage's connect and disconnect
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of connecting each stage one-by-one it is possible to write everything in single line of code using chaining:

.. code-block:: python

    >>> source = flow.add_stage('Dev Data Generator')
    >>> processor = flow.add_stage('Field Remover')
    >>> trash = flow.add_stage('Trash')
    >>> source.connect_output_to(processor).connect_output_to(trash)
    Trash_03(stage_id='Trash_03')

It is also possible to connect multiple stages to the output of a stage with a single call.

.. code-block:: python

    >>> source = flow.add_stage('Dev Data Generator')
    >>> renamer1 = flow.add_stage('Field Renamer')
    >>> renamer2 = flow.add_stage('Field Renamer')
    >>> source.connect_output_to(renamer1, renamer2)
    [FieldRenamer_01(stage_id='FieldRenamer_01'), FieldRenamer_02(stage_id='FieldRenamer_02')]


.. _preparing_data__streaming__editing_stage_configuration:

Editing a Stage's Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stages have properties which can be configured.
You can edit a stage's configuration through the :py:attr:`Stage.configuration <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.configuration>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.configuration.Configuration` object which encapsulates a stage's configuration.
You can print out the configuration and edit it similar to a :py:class:`dict`.

.. code-block:: python

    >>> dev_random_source.configuration['stop_after_first_batch']
    False
    >>> dev_random_source.configuration['stop_after_first_batch'] = True
