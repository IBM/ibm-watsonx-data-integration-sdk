.. _preparing_data__hierarchical_data:

Hierarchical Data Stage
=======================

The :py:class:`~ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage`
processes XML and JSON documents by running an internal pipeline of processing steps called an *assembly*.
Think of the assembly as a mini-flow that lives inside the stage: it has its own Input and Output steps,
and you connect processing steps between them — just as you connect stages inside a batch flow.

.. _preparing_data__hierarchical_data__assembly:

The Assembly
~~~~~~~~~~~~

Every Hierarchical Data stage has exactly one
:py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.assembly.Assembly`.
Accessing the :py:attr:`HierarchicalDataStage.assembly <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.assembly>`
property for the first time creates a fresh assembly with an
:py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.input_step.InputStep`
and an
:py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.output_step.OutputStep`
already connected to each other.

The simplest possible assembly passes data straight through without any processing step in between.
In that case you only need to call
:py:meth:`OutputStep.propagate() <ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.output_step.OutputStep.propagate>`
pointing at the Input step, which copies all columns from the incoming link schema:

.. skip: start 'requires a live project'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps import (
    ...     InputStep, OutputStep
    ... )
    >>>
    >>> flow = project.create_flow(name='xml_flow', flow_type='batch')
    >>> row_gen = flow.add_stage('Row Generator', 'row_gen')
    >>> hd_stage = flow.add_stage('Hierarchical Data', 'hd')
    >>> peek = flow.add_stage('Peek', 'peek')
    >>>
    >>> link_1 = row_gen.connect_output_to(hd_stage)
    >>> link_1.name = 'Link_1'
    >>> link_1_schema = link_1.create_schema()
    >>> link_1_schema.add_field('VARCHAR', 'name', length=100)
    >>> link_1_schema.add_field('INTEGER', 'age')
    >>>
    >>> link_2 = hd_stage.connect_output_to(peek)
    >>> link_2.name = 'Link_2'
    >>>
    >>> # Access the assembly — Input and Output steps are created automatically
    >>> assembly = hd_stage.assembly
    >>> isinstance(assembly.input_step, InputStep)
    True
    >>> isinstance(assembly.output_step, OutputStep)
    True
    >>>
    >>> # Propagate all columns from the incoming link straight to the output
    >>> assembly.output_step.propagate(assembly.input_step)
    >>>
    >>> project.update_flow(flow)
    <Response [201]>

.. skip: end

.. _preparing_data__hierarchical_data__adding_steps:

Adding Steps to the Assembly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use :py:meth:`Assembly.add_step() <ibm_watsonx_data_integration.services.datastage.models.hierarchical.assembly.Assembly.add_step>`
to insert a processing step into the assembly.
The method takes the step type as a string and an optional label, and returns the typed step instance.

Because the Input and Output steps are connected by default, you must:

1. **Disconnect** the default ``Input → Output`` link.
2. **Connect** the new step between them using ``connect_output_to()``.
3. Call :py:meth:`OutputStep.propagate() <ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.output_step.OutputStep.propagate>`
   passing the new step so the output columns are derived from it.

The following example adds a Sort step:

.. skip: start 'requires a live project'

.. code-block:: python

    >>> sort_step = assembly.add_step('Sort', 'Sort_1')
    >>> sort_step.configuration.list_to_sort = 'top/InputLinks/Link_1'
    >>> sort_step.configuration.add_key('age', 'DESC')
    >>>
    >>> # 1. Remove the default Input → Output connection
    >>> assembly.input_step.disconnect_output_from(assembly.output_step)
    >>>
    >>> # 2. Wire: Input → Sort → Output
    >>> link_3 = assembly.input_step.connect_output_to(sort_step)
    >>> link_3.name = 'Link_3'
    >>> link_4 = sort_step.connect_output_to(assembly.output_step)
    >>> link_4.name = 'Link_4'
    >>>
    >>> # 3. Propagate columns from the sort step to the output
    >>> assembly.output_step.propagate(source=sort_step)
    >>>
    >>> project.update_flow(flow)
    <Response [201]>

.. skip: end

.. _preparing_data__hierarchical_data__propagate:

Propagating Columns to the Output Step
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:py:meth:`OutputStep.propagate() <ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.output_step.OutputStep.propagate>`
copies columns from the given source to the Output step.
The ``source`` argument can be:

- **A step object** — the output path is derived automatically from the step's configuration.
- **A raw path string** such as ``'top/InputLinks/Link_1'`` — used when pointing directly at a list path.

The ``chunk_path`` keyword argument is used by XML and JSON Parser steps to identify the repeating
element in the schema. When omitted the deepest repeating element is auto-detected.

.. _preparing_data__hierarchical_data__finalize:

Finalizing the Assembly
~~~~~~~~~~~~~~~~~~~~~~~

Calling :py:meth:`HierarchicalDataStage.finalize_assembly() <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.finalize_assembly>`
serializes the in-memory assembly into the compressed stage parameters the platform stores.
This is called **automatically** by
:py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>`,
so you do not need to call it explicitly. The only reason to call it manually is to inspect the serialized
parameters before saving:

.. skip: start 'requires a live project'

.. code-block:: python

    >>> hd_stage.finalize_assembly()
    >>> hd_stage.configuration.e2_assembly is not None
    True

.. skip: end

.. _preparing_data__hierarchical_data__step_types:

Available Step Types
~~~~~~~~~~~~~~~~~~~~

The table below lists every step type string accepted by ``assembly.add_step()`` and the class it returns.

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Type string
     - Returned class
     - Purpose
   * - ``'Sort'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.sort_step.SortStep`
     - Sort a list of records by one or more fields
   * - ``'Aggregate'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.aggregate_step.AggregateStep`
     - Apply aggregate functions (Count, Sum, Min, Max, …) to a list
   * - ``'HJoin'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.hjoin_step.HJoinStep`
     - Hierarchical join — embed child records inside matching parent records
   * - ``'HPivot'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.hpivot_step.HPivotStep`
     - Pivot column values into child rows (horizontal → vertical)
   * - ``'VPivot'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.vpivot_step.VPivotStep`
     - Pivot categorised rows into columns (vertical → horizontal)
   * - ``'Regroup'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.regroup_step.RegroupStep`
     - Re-structure flat records into nested parent/child groups
   * - ``'Switch'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.switch_step.SwitchStep`
     - Route records into named branches based on conditions
   * - ``'OrderJoin'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.orderjoin_step.OrderJoinStep`
     - Merge two ordered lists by interleaving their records
   * - ``'JSONParser'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.json_parser_step.JSONParserStep`
     - Parse JSON input from a file, string set, or LOB column
   * - ``'JSONComposer'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.json_composer_step.JSONComposerStep`
     - Compose JSON output to a file, string set, or LOB column
   * - ``'XMLParser'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.xml_parser_step.XMLParserStep`
     - Parse XML input using an XSD schema from a schema library
   * - ``'XMLComposer'``
     - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.xml_composer_step.XMLComposerStep`
     - Compose XML output using an XSD schema from a schema library

.. _preparing_data__hierarchical_data__schema_libraries:

Attaching a Schema Library
~~~~~~~~~~~~~~~~~~~~~~~~~~

XML and JSON Parser/Composer steps require a
:py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema_library.SchemaLibrary`
to be attached to the stage. Use
:py:meth:`HierarchicalDataStage.add_schema_library() <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.add_schema_library>`:

.. skip: start 'requires a live project'

.. code-block:: python

    >>> hd_stage.add_schema_library(schema_lib)
    >>> project.update_flow(flow)
    <Response [200]>

.. skip: end

Duplicate entries are silently ignored. To detach a library use
:py:meth:`HierarchicalDataStage.remove_schema_library() <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.remove_schema_library>`.
For full details on creating and managing schema libraries, see
:ref:`XML Schema Libraries <preparing_data__schema_libraries>`.

.. seealso::
    - :ref:`Examples — Creating and Running a Hierarchical Data Flow with XML Parsing <preparing_data__examples__hierarchical_data_flow>` — Full end-to-end example
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage` — API reference for the stage
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.assembly.Assembly` — API reference for the Assembly
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.hierarchical.steps.output_step.OutputStep` — API reference for the Output step
    - :ref:`XML Schema Libraries <preparing_data__schema_libraries>` — Creating and managing schema libraries
