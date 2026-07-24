.. invisible-code-block: python

    >>> from ibm_watsonx_data_integration.common.constants import DeploymentEnvironment
    >>> project._platform._deployment_environment = DeploymentEnvironment.ON_PREM

.. _preparing_data__batch__wrapped_stages:

Wrapped Stages
==============

A **wrapped stage** is a reusable project asset that encapsulates an external executable command into a DataStage stage.
Once created and configured, a wrapped stage appears in the batch flow editor just like any built-in stage, letting you
invoke custom scripts or binaries as part of a flow.

.. note::

    Wrapped stages are an on-premises feature and require the platform to be configured for an
    on-premises deployment. Calling any wrapped stage method from a SaaS or AWS deployment
    raises a ``RuntimeError``.

Wrapped stage management follows a two-step process:

1. **Define the asset** — create and configure the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage` asset in the project (command, inputs, outputs, properties, environment variables, exit codes).
2. **Use it in a flow** — add the asset to a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow` as a stage node using :py:meth:`BatchFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.add_stage>`.

.. _preparing_data__batch__wrapped_stages__creating:

Creating a Wrapped Stage
~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`Project.create_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_wrapped_stage>` to create a new wrapped stage asset in the project.

.. code-block:: python

    >>> wrapped_stage = project.create_wrapped_stage(
    ...     name='My Custom Stage',
    ...     description='Executes a custom post-processing script',
    ... )

.. _preparing_data__batch__wrapped_stages__listing_and_retrieving:

Listing and Retrieving Wrapped Stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Access wrapped stages through the :py:attr:`Project.wrapped_stages <ibm_watsonx_data_integration.cpd_models.project_model.Project.wrapped_stages>` property,
which returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStages` collection.

.. code-block:: python

    >>> # List all wrapped stages
    >>> project.wrapped_stages
    [WrappedStage(...)]

    >>> # Filter by name
    >>> stages = project.wrapped_stages.get_all(name='My Custom Stage')

    >>> # Get a single stage by name
    >>> my_stage = project.wrapped_stages.get(name='My Custom Stage')

    >>> # Get the total count
    >>> len(project.wrapped_stages)
    1

.. _preparing_data__batch__wrapped_stages__configuring:

Configuring a Wrapped Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A wrapped stage asset has several configurable properties. After modifying any of them, call
:py:meth:`Project.update_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_wrapped_stage>` to persist the changes.

**Name and description**

.. code-block:: python

    >>> wrapped_stage.command = '/opt/scripts/my_script.sh'
    >>> wrapped_stage.description = 'Executes a custom post-processing script with configurable options'
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

**Execution mode**

The :py:attr:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.execution_mode` property controls whether the stage runs
in parallel (``'par'``) or sequential (``'seq'``) mode. The default is ``'par'``.

.. code-block:: python

    >>> wrapped_stage.execution_mode = 'seq'
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

.. _preparing_data__batch__wrapped_stages__inputs_outputs:

Adding Inputs and Outputs
~~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`WrappedStage.add_input() <ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_input>`
and :py:meth:`WrappedStage.add_output() <ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_output>`
to define the stage's data interfaces.

The ``stream`` parameter determines how data is passed between DataStage and the executable:

* Pass a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.FileDescriptor` enum value (``STDIN``, ``STDOUT``, or ``STDERR``) for **stream-based** links, where data is piped directly through standard file descriptors.
* Pass a plain **string** for **non-stream** links, where the string is the name of the argument or variable used to pass the file path to the executable. Use the ``is_command_line`` parameter to specify how the name is passed:

  * ``is_command_line=True`` (the default) — the name is passed as a **command-line argument**.
  * ``is_command_line=False`` — the name is passed as an **environment variable**.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage import FileDescriptor
    >>>
    >>> # Stream-based: data piped via stdin
    >>> wrapped_stage.add_input('input1', FileDescriptor.STDIN)
    WrappedStage(...)
    >>>
    >>> # Stream-based: data piped via stdout
    >>> wrapped_stage.add_output('output1', FileDescriptor.STDOUT)
    WrappedStage(...)
    >>>
    >>> # Non-stream: file path passed as a command-line argument named OUTPUT_FILE
    >>> wrapped_stage.add_output('output2', 'OUTPUT_FILE', is_command_line=True)
    WrappedStage(...)
    >>>
    >>> # Non-stream: file path passed as an environment variable named INPUT_PATH
    >>> wrapped_stage.add_input('input2', 'INPUT_PATH', named_pipe="my_pipe_name", is_command_line=False)
    WrappedStage(...)
    >>>
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

.. note::

    ``is_command_line`` and ``named_pipe`` are only valid when ``stream`` is a plain string. Passing them
    alongside a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.FileDescriptor`
    value raises a ``ValueError``.

You can optionally attach a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition`
to associate a schema with the link.

.. code-block:: python

    >>> data_def = project.data_definitions.get(name='data_definition_1')
    >>> wrapped_stage.add_output('output_with_schema', FileDescriptor.STDOUT, data_definition=data_def)
    WrappedStage(...)
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

.. _preparing_data__batch__wrapped_stages__properties:

Adding Properties
~~~~~~~~~~~~~~~~~

Properties let users supply runtime configuration values to the wrapped stage.
Use :py:meth:`WrappedStage.add_property() <ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_property>` to add a property.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage import DataType
    >>>
    >>> wrapped_stage.add_property(
    ...     name='output_path',
    ...     data_type=DataType.PATHNAME,
    ...     prompt='Output file path',
    ...     default_value='/tmp/output',
    ... )
    WrappedStage(...)
    >>> wrapped_stage.add_property(
    ...     name='record_count',
    ...     data_type=DataType.INTEGER,
    ...     prompt='Number of records to process',
    ...     default_value='100',
    ...     required=False,
    ... )
    WrappedStage(...)
    >>> wrapped_stage.add_property(
    ...     name='mode',
    ...     data_type=DataType.LIST,
    ...     prompt='Processing mode',
    ...     list_values=['fast', 'normal', 'safe'],
    ... )
    WrappedStage(...)
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

Available :py:class:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.DataType` values:

* ``DataType.STRING``
* ``DataType.INTEGER``
* ``DataType.FLOAT``
* ``DataType.PATHNAME``
* ``DataType.LIST``
* ``DataType.INPUTCOLUMN``
* ``DataType.BOOLEAN``

.. note::

    When using ``DataType.LIST``, the ``list_values`` parameter is required and must be a list of strings.
    If no ``default_value`` is provided, the first item in ``list_values`` is used as the default.

.. _preparing_data__batch__wrapped_stages__environment_variables:

Adding Environment Variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`WrappedStage.add_environment_variable() <ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_environment_variable>`
to declare environment variables that are set in the executable's environment at runtime.

.. code-block:: python

    >>> wrapped_stage.add_environment_variable('LOG_LEVEL', 'INFO')
    WrappedStage(...)
    >>> wrapped_stage.add_environment_variable('TEMP_DIR', '/tmp')
    WrappedStage(...)
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

.. _preparing_data__batch__wrapped_stages__exit_codes:

Configuring Exit Codes
~~~~~~~~~~~~~~~~~~~~~~~

You can add failure and success codes to the wrapped stage as well but you cannot add both at the same time. If ``all_exit_codes_successful`` is ``True`` (the default), use :py:meth:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_failure_code`.
If it is ``False``, use :py:meth:`~ibm_watsonx_data_integration.services.datastage.models.components.wrapped_stage.WrappedStage.add_success_code`.

**Failure codes**:

.. code-block:: python

    >>> wrapped_stage.add_failure_code('CODE_NAME_1')
    WrappedStage(...)
    >>> wrapped_stage.add_failure_code('255')
    WrappedStage(...)
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>

**Success codes**:

.. code-block:: python

    >>> wrapped_stage.entity.wrapped.environment.exit_codes.all_exit_codes_successful = False
    >>> wrapped_stage.add_success_code('SUCCESS_CODE_1')
    WrappedStage(...)
    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>


.. _preparing_data__batch__wrapped_stages__generating:

Generating a Wrapped Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before a wrapped stage can be used in a batch flow, it must be **generated**. Generation compiles the stage's
configuration into a usable DataStage stage type that the flow editor recognises. This is a separate step
from saving — updating the asset persists the configuration, but generation is what makes the stage
available for use in flows.

In the UI, these are two distinct buttons on the wrapped stage editor:

* **Save** — persists changes to the asset (equivalent to
  :py:meth:`Project.update_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_wrapped_stage>`).
* **Generate** — compiles the saved configuration into a usable stage type (equivalent to
  :py:meth:`Project.generate_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.generate_wrapped_stage>`).

.. image:: /_static/images/wrapped_stages/generate_wrapped_stage.png
    :alt: The Save and Generate buttons in the wrapped stage editor

In the SDK, call :py:meth:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.generate_wrapped_stage`
after saving your changes:

.. code-block:: python

    >>> project.update_wrapped_stage(wrapped_stage)
    <Response [200]>
    >>> project.generate_wrapped_stage(wrapped_stage)
    <Response [204]>

.. note::

    * You must generate a wrapped stage at least once before adding it to a flow. Re-generate after any
      configuration changes so that the flow editor picks up the latest definition.
    * The UI provides a **Replace** button that swaps the current wrapped stage asset with a different one.
      This operation is not yet implemented in the SDK.

.. _preparing_data__batch__wrapped_stages__using_in_flow:

Using a Wrapped Stage in a Batch Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once a wrapped stage asset exists in the project, add it to any batch flow using
:py:meth:`BatchFlow.add_stage() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.add_stage>`
with ``type='Wrapped Stage'`` and the ``wrapped_stage_name`` parameter set to the asset's name.

.. code-block:: python

    >>> stage_node = batch_flow.add_stage(
    ...     type='Wrapped Stage',
    ...     wrapped_stage_name='My Custom Stage',
    ... )
    >>> project.update_flow(batch_flow)
    <Response [201]>

You can also supply a custom ``label`` for the stage node within the flow:

.. code-block:: python

    >>> stage_node = batch_flow.add_stage(
    ...     type='Wrapped Stage',
    ...     label='Pre-processing Step',
    ...     wrapped_stage_name='My Custom Stage',
    ... )
    >>> project.update_flow(batch_flow)
    <Response [201]>

.. _preparing_data__batch__wrapped_stages__duplicating:

Duplicating a Wrapped Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`Project.duplicate_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_wrapped_stage>`
to create a copy of an existing wrapped stage asset. If no ``name`` is provided, the copy is named ``"<original name> cloned"``.

.. code-block:: python

    >>> cloned = project.duplicate_wrapped_stage(wrapped_stage=wrapped_stage, name='My Custom Stage v2')
    >>> cloned.name
    'My Custom Stage v2'

.. _preparing_data__batch__wrapped_stages__deleting:

Deleting a Wrapped Stage
~~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`Project.delete_wrapped_stage() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_wrapped_stage>`
to permanently remove a wrapped stage asset from the project.

.. code-block:: python

    >>> project.delete_wrapped_stage(cloned)
    <Response [200]>

.. invisible-code-block: python

    >>> project._platform._deployment_environment = DeploymentEnvironment.SAAS
