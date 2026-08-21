.. _preparing_data__udi__flows:

Flows
=====

UDI (Unstructured Data Integration) flows provide a Python-based orchestration framework for data integration pipelines.
Unlike traditional batch or streaming flows, UDI flows use a declarative pipeline definition with operators that are executed by a Python orchestrator.

The SDK provides the following functionality to interact with UDI flows:
    * Creating a flow
    * Retrieving flows
    * Updating a flow
    * Duplicating a flow
    * Deleting a flow
    * Running jobs

.. _preparing_data__udi__flows__creating_a_flow:

Creating a Flow
~~~~~~~~~~~~~~~

UDI flows are created by first creating an empty flow, then adding operators using the
:py:meth:`UDIFlow.add_operator() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.add_operator>` method.

In the SDK, you can create a UDI flow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_flow>` method.
You must specify ``flow_type='udi'``.

.. skip: start

.. code-block:: python

    >>> new_flow = project.create_flow(name='My UDI flow', description='optional description', flow_type='udi')
    >>> new_flow
    UDIFlow(name='My UDI flow', description='optional description', flow_id=..., pipeline_details=...)

.. skip: end

After creating a flow, you can add operators to build your data integration pipeline.

.. note::
    To add operators that require data assets (like ``ingest_cpd_assets``), you first need to upload assets to your project.

.. note::
    After adding operators, you must call :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>`
    to save the changes to the server.

.. _preparing_data__udi__flows__retrieving_flows:

Retrieving Flows
~~~~~~~~~~~~~~~~

UDI flows can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.flows <ibm_watsonx_data_integration.cpd_models.project_model.Project.flows>` property.
To retrieve only UDI flows, use the :py:meth:`Project.flows.get_all() <ibm_watsonx_data_integration.common.models.CollectionModel.get_all>` method with ``flow_type='udi'``.

.. skip: start

.. code-block:: python

    >>> project.flows
    [...]
    >>> udi_flows = project.flows.get_all(flow_type='udi')
    >>> udi_flows
    [..., UDIFlow(name='My UDI flow', description='optional description', flow_id=..., pipeline_details=...), ...]
    >>> my_flow = project.flows.get(name='My UDI flow')
    >>> my_flow.name
    'My UDI flow'

.. skip: end

.. _preparing_data__udi__flows__updating_a_flow:

Updating a Flow
~~~~~~~~~~~~~~~

You can update a UDI flow's name and description using the :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method.

.. skip: start

.. code-block:: python

    >>> new_flow.name = 'Updated UDI Flow Name'
    >>> new_flow.description = 'Updated description'
    >>> response = project.update_flow(new_flow)
    >>> response.status_code
    200

.. skip: end

.. _preparing_data__udi__flows__duplicating_a_flow:

Duplicating a Flow
~~~~~~~~~~~~~~~~~~

You can create a copy of an existing UDI flow using the project's ``duplicate_flow()`` method.
This creates a new flow with the same pipeline definition but with a new name and description.

.. skip: start

.. code-block:: python

    >>> udi_flow = project.flows.get(name='My UDI flow')
    >>> duplicated_flow = project.duplicate_flow(udi_flow, name='Copy of My Flow', description='Duplicated flow for testing')
    >>> duplicated_flow.name
    'Copy of My Flow'
    >>> project.delete_flow(duplicated_flow)
    <Response [...]>

.. skip: end

.. note::
    The ``duplicate_flow()`` method creates a copy of the flow by fetching its complete definition from the API and creating a new flow with the same configuration.

.. _preparing_data__udi__flows__deleting_a_flow:

Deleting a Flow
~~~~~~~~~~~~~~~

To delete a UDI flow, use the :py:meth:`Project.delete_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_flow>` method.

.. skip: start

.. code-block:: python

    >>> response = project.delete_flow(new_flow)
    >>> response.status_code in (200, 204)
    True

.. skip: end

.. _preparing_data__udi__flows__running_jobs:

Running Jobs
~~~~~~~~~~~~

UDI flows can be executed as jobs. Create and run a job using the :py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>` method.

.. skip: start

.. code-block:: python

    >>> job = project.create_job(flow=new_flow, name='My UDI Job', description='Process data with UDI pipeline')
    >>> job
    Job(..., name='My UDI Job', job_id=...)
    >>> job_run = job.start()
    >>> job_run.state
    '...'
    >>> project.delete_job(job)
    <Response [...]>

.. skip: end

.. _preparing_data__udi__flows__discovering_operators:

Discovering Available Operators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To discover all available operators and their parameters, use the
:py:attr:`UDIFlow.operators_metadata <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.operators_metadata>` property.
This returns a collection of :py:class:`~ibm_watsonx_data_integration.services.udi.models.operator_model.OperatorMetadata` entries,
each describing an operator type including required and optional parameters, parameter types, descriptions, and default values.

.. skip: start

.. code-block:: python

    >>> # Get metadata for all available operators
    >>> all_metadata = new_flow.operators_metadata.get_all()
    >>> all_metadata
    [..., OperatorMetadata(operator_type='ingest_cpd_assets', category=..., attributes=..., features=...), ...]
    >>> # View all available operator types
    >>> operator_types = [m.operator_type for m in all_metadata]
    >>> 'ingest_cpd_assets' in operator_types
    True
    >>> # Get metadata for a specific operator type
    >>> ingest_metadata = new_flow.operators_metadata.get(operator_type='ingest_cpd_assets')
    >>> ingest_metadata.operator_type
    'ingest_cpd_assets'

.. skip: end

.. _preparing_data__udi__flows__custom_operators:

Custom Operators
~~~~~~~~~~~~~~~~

UDI flows support custom operators that extend the built-in functionality with user-defined Python code.
Custom operators are stored in the project settings and can be listed, uploaded, and used in flows.

Listing Custom Operators
^^^^^^^^^^^^^^^^^^^^^^^^^

Custom operators are project-level resources that can be used across multiple flows.
To list all custom operators registered in your project, use the
:py:attr:`Project.custom_operators <ibm_watsonx_data_integration.cpd_models.project_model.Project.custom_operators>` property:

.. skip: start

.. code-block:: python

    >>> # List all custom operators in the project
    >>> for op in project.custom_operators:
    ...     print(f"{op.name}")
    hello_world

.. skip: end

The :py:class:`~ibm_watsonx_data_integration.services.udi.models.custom_operator_model.CustomOperators`
collection provides access to all custom operators with support for filtering and iteration.

Uploading Custom Operators
^^^^^^^^^^^^^^^^^^^^^^^^^^^

To upload a custom operator to your project, use the
:py:meth:`Project.upload_custom_operator() <ibm_watsonx_data_integration.cpd_models.project_model.Project.upload_custom_operator>` method:

.. skip: start

.. code-block:: python

    >>> # Upload a custom operator file (requires actual file)
    >>> custom_op = project.upload_custom_operator(
    ...     name='my_custom_operator',
    ...     file_path='/path/to/operator.py',
    ...     dependency='/path/to/dependencies.tar.gz'  # Optional
    ... )
    >>> custom_op.name
    'my_custom_operator'

.. skip: end

Deleting Custom Operators
^^^^^^^^^^^^^^^^^^^^^^^^^^

To delete a custom operator from your project, use the
:py:meth:`Project.delete_custom_operator() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_custom_operator>` method:

.. skip: start

.. code-block:: python

    >>> # Delete a custom operator
    >>> custom_op = project.custom_operators.get(name='my_custom_operator')
    >>> project.delete_custom_operator(custom_op)
    {'status_code': 200}

.. skip: end

Using Custom Operators
^^^^^^^^^^^^^^^^^^^^^^^

Once a custom operator is uploaded to your project, you can use it in your flow like any built-in operator:

.. skip: start

.. code-block:: python

    >>> # Add a custom operator to your flow
    >>> custom_op = my_flow.add_operator('my_custom_operator', param1='value1')
    >>> custom_op
    <BaseOperator(type='my_custom_operator', ...)>

.. skip: end

.. _preparing_data__udi__flows__deployment_spaces:

Deployment Spaces
~~~~~~~~~~~~~~~~~

UDI flows can be promoted from projects to deployment spaces for production use.

Listing Available Spaces
^^^^^^^^^^^^^^^^^^^^^^^^^

Spaces are platform-level resources (similar to projects) that serve as deployment targets for flows.
To list all available deployment spaces, use the
:py:attr:`Platform.spaces <ibm_watsonx_data_integration.platform.Platform.spaces>` property:

.. skip: start

.. code-block:: python

    >>> # List all available deployment spaces
    >>> for space in platform.spaces:
    ...     print(f"{space.name}: {space.space_id}")
    Production Space: 12345-abcde-67890
    Development Space: 98765-fghij-43210
    >>> # Get specific space by ID
    >>> prod_space = platform.spaces['12345-abcde-67890']

.. skip: end

Promoting to a Space
^^^^^^^^^^^^^^^^^^^^

To promote a flow from a project to a deployment space, use the
:py:meth:`UDIFlow.promote_to_space() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.promote_to_space>` method:

.. skip: start

.. code-block:: python

    >>> # Promote flow to a deployment space
    >>> promotion_response = my_flow.promote_to_space(
    ...     target_space_id='12345-abcde-67890',
    ...     flow_name='Production Flow',
    ...     description='Flow promoted to production',
    ...     duplicate_action='REPLACE'  # or 'IGNORE' to skip if exists
    ... )
    >>> promotion_response['flow_id']
    'promoted-flow-id-123'

.. skip: end

Running in a Space
^^^^^^^^^^^^^^^^^^

After promoting a flow to a space, you need to work with the space directly to create and run jobs.
The promotion response contains the ``flow_id`` and ``space_id`` needed to interact with the space-based flow.

.. note::
    Space-based jobs are managed separately from project-based jobs. You should use the space's job management
    APIs directly rather than mixing project and space contexts.

.. _preparing_data__udi__flows__execution_logs:

Execution Logs
~~~~~~~~~~~~~~

To retrieve execution logs for a flow run, use the
:py:meth:`UDIFlow.get_execution_logs() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.get_execution_logs>` method:

.. skip: start

.. code-block:: python

    >>> # Get logs for a specific job run
    >>> logs = my_flow.get_execution_logs(run_id='job-run-id-123')
    >>> for log_entry in logs:
    ...     print(log_entry)
    [INFO] Starting flow execution...
    [INFO] Processing operator: ingest_cpd_assets
    [INFO] Flow execution completed successfully

.. skip: end

.. _preparing_data__udi__flows__branching_and_merging:

Branching and Merging
~~~~~~~~~~~~~~~~~~~~~

UDI flows support conditional branching and merging, allowing you to create complex data processing pipelines
that route data through different processing paths based on conditions.

Creating a Branching Operator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To create a branching operator, use the :py:meth:`UDIFlow.add_operator() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.add_operator>`
method with ``operator_type='branching'``. This returns a :py:class:`~ibm_watsonx_data_integration.services.udi.models.branch_model.BranchingOperator`
instance that allows you to add conditional branches.

.. skip: start

.. code-block:: python

    >>> # Create a branching operator
    >>> quality_branch = my_flow.add_operator('branching', name='Quality-Based Processing')
    >>> quality_branch
    BranchingOperator(name='Quality-Based Processing', branches=0)

.. skip: end

Adding Branches
^^^^^^^^^^^^^^^

Use the :py:meth:`BranchingOperator.add_branch() <ibm_watsonx_data_integration.services.udi.models.branch_model.BranchingOperator.add_branch>`
method to add conditional branches. Each branch requires a condition expression and can optionally have a label and merge link name.

.. skip: start

.. code-block:: python

    >>> # Add a high quality branch
    >>> high_quality = quality_branch.add_branch(
    ...     condition='docq_total_words >= 100',
    ...     label='High Quality',
    ...     merge_link_name='high_quality_link'
    ... )
    >>> # Add operators to the high quality branch
    >>> high_quality.add_operator('chunker', chunk_size=1000, chunk_overlap=200)
    <BaseOperator(type='chunker', ...)>
    >>> high_quality.add_operator('embeddings')
    <BaseOperator(type='embeddings', ...)>
    >>> # Add a low quality branch
    >>> low_quality = quality_branch.add_branch(
    ...     condition='docq_total_words < 100',
    ...     label='Low Quality',
    ...     merge_link_name='low_quality_link'
    ... )
    >>> # Add operators to the low quality branch
    >>> low_quality.add_operator('doc_quality')
    <BaseOperator(type='doc_quality', ...)>
    >>> low_quality.add_operator('chunker', chunk_size=500, chunk_overlap=100)
    <BaseOperator(type='chunker', ...)>

.. skip: end

Merging Branches
^^^^^^^^^^^^^^^^

After branching, you can merge the branches back together using a merge operator. The merge operator combines
data from multiple branches into a single stream.

.. skip: start

.. code-block:: python

    >>> # Add a merge operator to combine branches
    >>> merge_op = my_flow.add_operator('merge', merge_type='rows')
    >>> merge_op
    <BaseOperator(type='merge', ...)>
    >>> # Continue processing after merge
    >>> my_flow.add_operator('milvusdb_cp4d', connection=milvus_conn, collection_name='documents')
    <BaseOperator(type='milvusdb_cp4d', ...)>

.. skip: end

Nested Branching
^^^^^^^^^^^^^^^^

Branches can contain nested branching operators, allowing for complex multi-level conditional logic.

.. skip: start

.. code-block:: python

    >>> # Create main branching operator
    >>> main_branch = my_flow.add_operator('branching', name='Main Quality Branch')
    >>> # Add high quality branch
    >>> high_quality = main_branch.add_branch(condition='quality_score >= 0.8', label='High Quality')
    >>> high_quality.add_operator('chunker', chunk_size=1000)
    <BaseOperator(type='chunker', ...)>
    >>> # Add nested branching within high quality branch
    >>> content_type_branch = high_quality.add_branching(name='Content Type Branch')
    >>> expense_branch = content_type_branch.add_branch(condition='type == \'expense\'')
    >>> expense_branch.add_operator('embeddings')
    <BaseOperator(type='embeddings', ...)>
    >>> invoice_branch = content_type_branch.add_branch(condition='type == \'invoice\'')
    >>> invoice_branch.add_operator('doc_quality')
    <BaseOperator(type='doc_quality', ...)>

.. skip: end

Complete Branching Example
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here's a complete example showing a UDI flow with branching and merging:

.. skip: start

.. code-block:: python

    >>> # Create flow
    >>> branching_flow = project.create_flow(name='Branching Flow', flow_type='udi')
    >>> # Add initial operators
    >>> ingest_op = branching_flow.add_operator('ingest_cpd_assets', data_assets=asset_ids)
    >>> extract_op = branching_flow.add_operator('extract_cpd', ocr_mode='disabled')
    >>> doc_quality_op = branching_flow.add_operator('doc_quality')
    >>> # Create branching operator
    >>> quality_branch = branching_flow.add_operator('branching', name='Quality-Based Processing')
    >>> # High quality branch
    >>> high_quality = quality_branch.add_branch(
    ...     condition='docq_total_words >= 100',
    ...     label='High Quality',
    ...     merge_link_name='high_quality_link'
    ... )
    >>> high_quality.add_operator('chunker', chunk_size=1000, chunk_overlap=200, chunk_type='watsonx')
    <BaseOperator(type='chunker', ...)>
    >>> high_quality.add_operator('embeddings', name='High Quality Embeddings')
    <BaseOperator(type='embeddings', ...)>
    >>> # Low quality branch
    >>> low_quality = quality_branch.add_branch(
    ...     condition='docq_total_words < 100',
    ...     label='Low Quality',
    ...     merge_link_name='low_quality_link'
    ... )
    >>> low_quality.add_operator('doc_quality', name='Re-check Quality')
    <BaseOperator(type='doc_quality', ...)>
    >>> low_quality.add_operator('chunker', chunk_size=500, chunk_overlap=100)
    <BaseOperator(type='chunker', ...)>
    >>> low_quality.add_operator('embeddings', name='Low Quality Embeddings')
    <BaseOperator(type='embeddings', ...)>
    >>> # Merge branches
    >>> merge_op = branching_flow.add_operator('merge', merge_type='rows')
    >>> # Save the flow
    >>> project.update_flow(branching_flow)
    <Response [200]>
    >>> # Clean up
    >>> project.delete_flow(branching_flow)
    <Response [...]>

.. skip: end

.. note::
    - Branch conditions use feature names from upstream operators (e.g., ``docq_total_words`` from ``doc_quality`` operator)
    - The ``merge_link_name`` parameter helps identify which branch data came from when merging
    - Branches are evaluated in the order they are added
    - Each branch can have its own sequence of operators

.. _preparing_data__udi__flows__parameter_sets:

Parameter Sets and Local Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

UDI flows support parameter sets and local parameters — letting you pass values into
operator fields at run time rather than hardcoding them into the flow definition.

.. note::
    Parameter sets are project-level assets shared across flows.
    See :ref:`preparing_data__parameter_sets` for how to create and manage them.

Attaching a Parameter Set to a Flow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you have a parameter set, attach it to a UDI flow with
:py:meth:`UDIFlow.use_parameter_set() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.use_parameter_set>`:

.. skip: start

.. code-block:: python

    >>> from ibm_watsonx_data_integration.cpd_models.parameter_set_model import ParameterType
    >>> paramset = project.create_parameter_set('udi_params')
    >>> paramset.add_parameter(parameter_type=ParameterType.String, name='collection_name', value='my_collection')
    ParameterSet(name='udi_params', ...)
    >>> project.update_parameter_set(paramset)
    <Response [200]>
    >>> my_flow.use_parameter_set(paramset)
    UDIFlow(...)

.. skip: end

To reference the parameter value inside an operator field, use the ``#ParameterSetName.parameter_name#`` notation:

.. skip: start

.. code-block:: python

    >>> milvus_op = my_flow.add_operator('milvusdb_cp4d')
    >>> milvus_op.collection_name = '#udi_params.collection_name#'
    >>> project.update_flow(my_flow)
    <Response [200]>

.. skip: end

Selecting a Default Value Set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If your parameter set has multiple value sets, set which one the flow uses by default with
:py:meth:`UDIFlow.set_runtime_value_set() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.set_runtime_value_set>`:

.. skip: start

.. code-block:: python

    >>> my_flow.set_runtime_value_set(parameter_set_name='udi_params', value_set_name='production')
    UDIFlow(...)

.. skip: end

Overriding a Parameter Set Value at the Flow Level
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To override a specific parameter value as a flow-level default (affects all jobs created from this flow),
use :py:meth:`UDIFlow.set_runtime_parameter_value() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.set_runtime_parameter_value>`:

.. skip: start

.. code-block:: python

    >>> my_flow.set_runtime_parameter_value(
    ...     parameter_set_name='udi_params',
    ...     parameter_name='collection_name',
    ...     value='staging_collection'
    ... )
    UDIFlow(...)

.. skip: end

.. note::
    To override parameter values for a specific job or job run only, pass ``runtime_parameters``
    to :py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>`
    or :py:meth:`Job.start() <ibm_watsonx_data_integration.cpd_models.job_model.job_model.Job.start>`
    instead. See `Per-Run Overrides via Job Start`_ below.

Adding Local Parameters
^^^^^^^^^^^^^^^^^^^^^^^^

Local parameters are flow-scoped — they exist only within a single flow and do not require a
parameter set asset. Add one with
:py:meth:`UDIFlow.add_local_parameter() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.add_local_parameter>`:

.. skip: start

.. code-block:: python

    >>> my_flow.add_local_parameter(
    ...     parameter_type=ParameterType.String,
    ...     name='chunk_size',
    ...     value='1000',
    ...     description='Default chunk size for chunker operators'
    ... )
    UDIFlow(...)

.. skip: end

Pass the parameter's default value directly when adding the operator.
The engine uses ``configuration.parameters.local_parameters`` to override the value
at run time — no token placeholder is stored in the operator field:

.. skip: start

.. code-block:: python

    >>> chunker_op = my_flow.add_operator('chunker', chunk_size='1000')
    >>> project.update_flow(my_flow)
    <Response [200]>

.. skip: end

Overriding a Local Parameter Value at Run Time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use
:py:meth:`UDIFlow.set_runtime_local_parameter() <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.set_runtime_local_parameter>`
to change a local parameter value for all subsequent job runs on this flow instance:

.. skip: start

.. code-block:: python

    >>> my_flow.set_runtime_local_parameter(local_parameter_name='chunk_size', value='500')
    UDIFlow(...)

.. skip: end

Per-Run Overrides via Job Start
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To override parameter values at the job level (all runs of that job) or for a single run only,
pass ``runtime_parameters`` to
:py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>`
or :py:meth:`Job.start() <ibm_watsonx_data_integration.cpd_models.job_model.job_model.Job.start>`.
This follows the same pattern used by batch and streaming flows:

.. skip: start

.. code-block:: python

    >>> # Override at job level — applies to all runs of this job
    >>> job = project.create_job(flow=my_flow, name='UDI Run', runtime_parameters={'udi_params.collection_name': 'prod_collection'})
    >>> # Override at run level — applies to this run only
    >>> job_run = job.start(runtime_parameters={'udi_params.collection_name': 'override_collection'})
    >>> # Override a local parameter: bare name
    >>> job_run = job.start(runtime_parameters={'chunk_size': '2000'})

.. skip: end

.. _preparing_data__udi__flows__pipeline_operators:

Pipeline Operators
~~~~~~~~~~~~~~~~~~

UDI flows support various operators for data processing:

* **ingest_cpd_assets**: Ingest assets from Cloud Pak for Data
* **extract_cpd**: Extract data from CPD assets
* **chunker**: Split data into chunks for processing
* **pii_and_hap_extract_redact**: Detect and redact PII/HAP
* **embeddings**: Generate embeddings for text data
* **milvusdb_cp4d**: Store data in Milvus vector database
* **branching**: Create conditional branches for routing data based on conditions
* **merge**: Merge multiple branches back into a single stream

Use :py:attr:`UDIFlow.operators_metadata <ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow.operators_metadata>`
to discover all available operators and their parameters dynamically.

For detailed information about branching and merging, see :ref:`preparing_data__udi__flows__branching_and_merging`.



.. _preparing_data__udi__flows__api_reference:

API Reference
~~~~~~~~~~~~~

For detailed API documentation, see:

* :py:class:`~ibm_watsonx_data_integration.services.udi.models.flow_model.UDIFlow`
* :py:class:`~ibm_watsonx_data_integration.services.udi.api.flow_api.UDIFlowApiClient`
