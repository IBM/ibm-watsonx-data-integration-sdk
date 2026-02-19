.. _preparing_data__batch_stages:

Stages
======

.. include:: ../shared/stages/introduction.rst

The SDK provides many ways to interact between a flow and its stages:
    * Adding a stage to a flow
    * Connecting stages
    * Editing a stage's configuration

.. _preparing_data__batch__listing_stages_in_flow:

Listing and retreiving Stages in a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all the stages in a flow, you can use the :py:attr:`BatchFlow.stages <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.stages>` property. This returns a :py:class:`list` of :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode` instances.

.. code-block:: python

    >>> batch_flow.stages
    [RowGeneratorStage(label='Row_Generator'), PeekStage(label='Peek'), RowGeneratorStage(label='Row_Generator_2'), PeekStage(label='Peek_2'), GoogleBigqueryStage(label='BigQuery')]

If you wish to filter these stages using arguments you can use the :py:meth:`Stages.get_all() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.stages.get_all>` method.
These arguments can be any of the fields found in the configurations of the stages you want, such as the ``type`` of the stage or the ``output_count`` of the stage.

.. code-block:: python

    >>> batch_flow.stages.get_all(type="Row Generator")
    [RowGeneratorStage(label='Row_Generator'), RowGeneratorStage(label='Row_Generator_2')]

Finally, to retreive a specific stage from a flow use the :py:meth:`Stages.get() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.stages.get>` method.
The ``label`` of a stage is a unique identifier so you can pass this value in to find the desired stage.

.. code-block:: python

    >>> batch_flow.stages.get(label="Row_Generator")
    RowGeneratorStage(label='Row_Generator')

.. _preparing_data__batch__add_stage_to_flow:

Adding a Stage to a Flow
~~~~~~~~~~~~~~~~~~~~~~~~

To add stages to a flow in the UI, you can open the dropdown menu on the left of the flow edit page and select the stage you want to add.

.. image:: /_static/images/stages/add_batch_stage.png
   :alt: Screenshot for adding a stage.
   :align: center
   :width: 40%

In the SDK, you can use the :py:meth:`BatchFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.add_stage>` method.
This method takes in the stage's official ``type`` and the ``label`` for that stage. The ``type`` is the unique identifier for that stage and the ``label`` is the name you wish to give it.

This method returns the newly created stage.

.. code-block:: python

    >>> amazon_rds = batch_flow.add_stage(type = 'Amazon RDS for PostgreSQL', label = 'Amazon RDS')
    >>> project.update_flow(batch_flow)
    <Response [201]>

You can find a list of all unique identifiers here: :py:meth:`BatchFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.add_stage>`

.. _preparing_data__batch__remove_stage_from_flow:

Remove a Stage from a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can click on the three dots at the top right of the stage and click on the delete button from the menu.

.. image:: /_static/images/stages/remove_batch_stage.png
   :alt: Screenshot for adding a stage.
   :align: center
   :width: 40%


In the SDK, you can remove a stage from a flow using the :py:meth:`BatchFlow.remove_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.remove_stage>` method
and passing an instance of :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode` to it.

All stages connected to this stage will be disconnected by this action.

This method does not return anything.

After removing stages you need to call :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method to save the changes.

.. code-block:: python

    >>> amazon_sqs_consumer = batch_flow.stages.get(label="Amazon RDS")
    >>> batch_flow.remove_stage(amazon_sqs_consumer)
    >>> project.update_flow(flow)
    <Response [200]>

.. _preparing_data__batch__listing_connected_links:

Listing all Links connected to a Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the SDK, you can list all the links connected to a stage by using the :py:meth:`StageNode.get_input_links() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode.get_input_links>` and :py:meth:`StageNode.get_output_links() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode.get_output_links>` methods.
Batch_flow currently has a ``Row Generator Stage`` connected to a ``Peek Stage``.

.. code-block:: python

    >>> Row_Generator_Stage = batch_flow.stages.get(label="Row_Generator")
    >>> Row_Generator_Stage.get_output_links()
    [Link_1 (src='Row_Generator', dest='Peek')]
    >>> Peek_Stage = batch_flow.stages.get(label="Peek")
    >>> Peek_Stage.get_input_links()
    [Link_1 (src='Row_Generator', dest='Peek')]


.. _preparing_data__batch__connecting_stages:

Connecting Stages
~~~~~~~~~~~~~~~~~

In the UI, to connect stages, you can click on the output of a stage and drag it to another stage.

.. image:: /_static/images/stages/connect_batch_stages.png
   :alt: Screenshot for connecting a stage.
   :align: center
   :width: 85%

In the SDK, to connect batch stages to each other we can use the :py:meth:`Stage.connect_output_to() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode.connect_output_to>` method.
This method connects two stages and returns the created link between them.

.. code-block:: python

    >>> row_gen = batch_flow.add_stage('Row Generator', 'Row_Generator') # a sample origin stage that generates data
    >>> peek = batch_flow.add_stage('Peek', 'Peek') # a sample destination stage that outputs all input to console
    >>> link = row_gen.connect_output_to(peek)


.. _preparing_data__batch__editing_stage_configuration:

Editing a Stage's Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All stages have properties which can be configured.
You can edit a stage's configuration through its corresponding ``configuration`` property.
This property returns an object containing all of that stage's properties. The configuration object supports both dot and bracket notation for editing.

.. Furthermore, the types of the properties depend on the types in the UI. For example, integer parameters will be integers where possible, checkboxes will be booleans where possible, and choices like dropdowns turn into enums.

.. code-block:: python

    >>> row_gen.configuration
    {
        "buf_mode": "default",
        "max_mem_buf_size": 3145728,
        ...
    }
    >>> row_gen.configuration.records = 5
    >>> row_gen.configuration["runtime_column_propagation"] = False
