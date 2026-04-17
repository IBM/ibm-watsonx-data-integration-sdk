.. _preparing_data__batch__compilation_mode:

Compilation Modes
=================

When creating or working with batch flows, you can specify the compilation mode that determines how the flow is executed. The SDK supports three compilation modes: **ETL**, **TETL**, and **ELT**.

This guide demonstrates how to use compilation modes when creating batch flows and how to change modes on existing flows.

.. _preparing_data__batch__compilation_mode__prerequisites:

Prerequisites
~~~~~~~~~~~~~

To use compilation modes with the SDK, make sure you have completed the following steps:
    * :ref:`Created a project<projects__projects__creating_a_project>`.
    * :ref:`Created a batch flow<preparing_data__batch__flows__creating_a_flow>`.

.. _preparing_data__batch__compilation_mode__overview:

Compilation Modes Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.CompileMode` enum defines the execution mode for DataStage batch flows:

**ETL (Extract, Transform, Load)** - Default mode
    A traditional ETL pattern in which data is extracted from sources, transformed in the DataStage engine, and then loaded into targets. This is the default compilation mode when creating a new batch flow.

**TETL (Transform, Extract, Transform, Load)**
    An enhanced ETL mode that performs transformations both before extraction and after loading, providing additional flexibility for complex data-processing scenarios.

**ELT (Extract, Load, Transform)**
    A modern pattern that uses SQL pushdown capabilities to move transformations to the target database. This mode is beneficial when working with powerful database systems that can efficiently execute the transformation logic.

.. _preparing_data__batch__compilation_mode__setting:

Setting Compilation Mode
~~~~~~~~~~~~~~~~~~~~~~~~

You can set the compilation mode on a flow by assigning to the ``compile_mode`` property and then calling
:py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` to persist the change.

.. invisible-code-block: python

    >>> from ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow import CompileMode, ELTMaterializationPolicy
    >>> elt_batch_flow = project.create_flow(name='my_elt_flow', flow_type='batch')
    >>> tetl_flow = project.create_flow(name='my_tetl_flow', flow_type='batch')

.. code-block:: python

    >>> # Set ELT compilation mode on a flow
    >>> elt_batch_flow.compile_mode = CompileMode.ELT
    >>> project.update_flow(elt_batch_flow)
    <Response [201]>

    >>> # Set TETL mode on a flow
    >>> tetl_flow.compile_mode = CompileMode.TETL
    >>> project.update_flow(tetl_flow)
    <Response [201]>

    >>> # Default mode is ETL if not specified
    >>> default_flow = project.create_flow(name='my_default_flow', flow_type='batch')
    >>> default_flow.compile_mode
    <CompileMode.ETL: 'ETL'>

.. _preparing_data__batch__compilation_mode__changing:

Changing Compilation Mode on Existing Flows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can retrieve an existing flow and modify its compilation mode:

.. invisible-code-block: python

    >>> existing_flow = project.create_flow(name='my_existing_flow', flow_type='batch')

.. code-block:: python

    >>> # Get an existing flow
    >>> existing_flow = project.flows.get(name='my_existing_flow')
    >>>
    >>> # Change the compilation mode
    >>> existing_flow.compile_mode = CompileMode.ELT
    >>>
    >>> # Persist the change to the project
    >>> project.update_flow(existing_flow)
    <Response [201]>

.. _preparing_data__batch__compilation_mode__elt_policies:

ELT Materialization Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When using **ELT mode**, you can further control the optimization strategy by setting an ELT materialization policy. The :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.ELTMaterializationPolicy` enum defines how SQL pushdown optimizations are handled:

**NESTED_QUERY** - Generate nested SQL (default for ELT mode)
    Generates nested SQL queries to minimize intermediate storage. This is the most efficient option for databases that handle complex nested queries well.

**INTERMEDIATE_TABLE** - Link as table
    Materializes intermediate results as temporary tables. This is useful when nested queries become too complex or when the database optimizer benefits from materialized intermediates.

**INTERMEDIATE_VIEW** - Link as view
    Creates intermediate views instead of tables. This provides a balance between nested queries and materialized tables.

**MATERIALIZE_CARDINALITY_CHANGERS** - Advanced
    An advanced optimization that selectively materializes stages that change cardinality (for example, aggregations and joins). This provides fine-grained control over what gets materialized.

.. note::

   ELT materialization policies can **only** be set when the flow's ``compile_mode`` is ``CompileMode.ELT``. Attempting to set this property in any other mode will raise a ``RuntimeError``.

.. _preparing_data__batch__compilation_mode__elt_with_policies:

Using ELT with Materialization Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. invisible-code-block: python

    >>> elt_flow = project.create_flow(name='elt_optimized_flow', flow_type='batch')
    >>> elt_flow.compile_mode = CompileMode.ELT

.. code-block:: python

    >>> # Set a specific materialization policy on an ELT flow
    >>> elt_flow.elt_materialization_policy = ELTMaterializationPolicy.INTERMEDIATE_TABLE
    >>> project.update_flow(elt_flow)
    <Response [201]>

    >>> # Default policy is NESTED_QUERY when switching to ELT mode
    >>> flow_with_defaults = project.create_flow(name='elt_defaults_flow', flow_type='batch')
    >>> flow_with_defaults.compile_mode = CompileMode.ELT
    >>> flow_with_defaults.elt_materialization_policy
    <ELTMaterializationPolicy.NESTED_QUERY: ('Generate nested SQL', 'NESTED_QUERY')>

    >>> # Change the materialization policy on an existing ELT flow
    >>> elt_flow.elt_materialization_policy = ELTMaterializationPolicy.INTERMEDIATE_VIEW
    >>> project.update_flow(elt_flow)
    <Response [201]>

.. _preparing_data__batch__compilation_mode__choosing:

Choosing the Right Compilation Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Consider these factors when selecting a compilation mode:

**Use ETL mode when:**
    - You need traditional DataStage transformation capabilities.
    - Data sources and targets are heterogeneous.
    - Complex transformations require DataStage-specific features.
    - This is the default and most widely compatible mode.

**Use TETL mode when:**
    - You need additional transformation flexibility.
    - Your workflow benefits from pre-extraction transformations.
    - You require enhanced data processing capabilities.

**Use ELT mode when:**
    - Your target database has powerful SQL optimization capabilities.
    - Data sources and targets support SQL pushdown.
    - You want to leverage database engine performance.
    - Network bandwidth between DataStage and the database is limited.
    - Large data volumes benefit from database-side processing.

**ELT Materialization Policy Selection:**
    - **NESTED_QUERY**: Best for databases with strong query optimizers (e.g., modern cloud databases).
    - **INTERMEDIATE_TABLE**: Use when nested queries are too complex or slow.
    - **INTERMEDIATE_VIEW**: Balance between query complexity and materialization overhead.
    - **MATERIALIZE_CARDINALITY_CHANGERS**: Advanced users who need fine-grained control.

.. _preparing_data__batch__compilation_mode__example:

Complete Example
~~~~~~~~~~~~~~~~

The following example shows how to create and work with a flow using compilation modes:

.. invisible-code-block: python

    >>> complete_flow = project.create_flow(name='elt_optimized_complete_flow', flow_type='batch')

.. code-block:: python

    >>> # Create a batch flow
    >>> complete_flow = project.create_flow(name='elt_optimized_complete_flow_2', flow_type='batch')
    >>>
    >>> # Set ELT mode for database pushdown
    >>> complete_flow.compile_mode = CompileMode.ELT
    >>>
    >>> # Set the materialization policy
    >>> complete_flow.elt_materialization_policy = ELTMaterializationPolicy.NESTED_QUERY
    >>> project.update_flow(complete_flow)
    <Response [201]>
    >>>
    >>> # Switch to ETL mode if needed
    >>> complete_flow.compile_mode = CompileMode.ETL
    >>> project.update_flow(complete_flow)
    <Response [201]>
