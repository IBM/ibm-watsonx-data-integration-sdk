.. _preparing_data__pipelines__components:

Pipeline Components
===================

Pipeline components are reusable building blocks that can be used within pipelines to perform specific tasks. The platform provides built-in components for common operations, and you can also create custom components for specialized functionality.

.. _preparing_data__pipelines__components__overview:

Overview
~~~~~~~~

Pipeline components encapsulate functionality that can be called within a pipeline definition. Each component has:

* A unique component ID
* Input parameters that define what data the component needs
* Output parameters that define what data the component produces

Components can be either:

* **Built-in components**: Provided by the platform for common tasks
* **Custom components**: Created by users for specialized functionality

.. _preparing_data__pipelines__components__listing:

Listing Pipeline Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the SDK, pipeline components can be retrieved through the :py:attr:`Project.pipeline_components <ibm_watsonx_data_integration.cpd_models.project_model.Project.pipeline_components>` property.

.. code-block:: python

    >>> # List only built-in components
    >>> project.pipeline_components.get_all(is_built_in=True)
    [PipelineComponent(...), ...]

    >>> # List only custom components
    >>> project.pipeline_components.get_all(is_built_in=False)  # doctest: +SKIP
    [PipelineComponent(component_id='...', name='My Custom Component', description=None, is_built_in=False), ...]

.. _preparing_data__pipelines__components__using_component_id_enum:

Using the PipelineComponentId Enum
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The SDK provides a :py:class:`~ibm_watsonx_data_integration.services.pipelines.models.component_ids.PipelineComponentId` enum that contains all built-in component IDs. Using this enum helps prevent typos and makes it easier to discover available components.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> # Get a component using the enum
    >>> run_datastage_job = project.pipeline_components.get(
    ...     component_id=PipelineComponentId.RUN_DATASTAGE_JOB
    ... )
    >>> run_datastage_job
    PipelineComponent(component_id='run-datastage-job', name='Run DataStage job', description='Run the DataStage job in the project or deployment space', is_built_in=True)

    >>> # Get a component by name
    >>> run_bash = project.pipeline_components.get(name='Run Bash script')
    >>> run_bash
    PipelineComponent(component_id='run-bash-script', name='Run Bash script', description='Run Bash script.', is_built_in=True)

.. _preparing_data__pipelines__components__built_in:

Built-in Components
~~~~~~~~~~~~~~~~~~~

The following sections describe each built-in component in detail, including what it does, its parameters, and usage examples.

.. _preparing_data__pipelines__components__run_datastage_job:

Run DataStage Job
^^^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.RUN_DATASTAGE_JOB``

**Description**: Executes a batch flow job within a pipeline. This component allows you to orchestrate DataStage jobs as part of a larger pipeline workflow.

**Parameters**:

* ``job`` (required): The DataStage job to execute. Can be a Job object or a job reference string.
* ``job_parameters`` (optional): Dictionary of job parameters to pass to the DataStage job.
* ``env_variables`` (optional): Dictionary of environment variables to set for the job run.

**Outputs**:

* Job run information and status

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> # Use the batch_flow fixture which already has stages configured
    >>> datastage_job = project.create_job(name='My DataStage Job', flow=batch_flow)
    >>>
    >>> @dsl.pipeline
    ... def my_pipeline() -> None:
    ...     # Get the component
    ...     run_ds_job = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_DATASTAGE_JOB
    ...     )
    ...
    ...     # Execute the DataStage job
    ...     run_ds_job(job=datastage_job)
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='Pipeline with DataStage',
    ...     pipeline_function=my_pipeline
    ... )

.. skip: end

**Example with Parameters**:

.. skip: start

.. code-block:: python

    >>> @dsl.pipeline
    ... def my_pipeline_with_params() -> None:
    ...     run_ds_job = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_DATASTAGE_JOB
    ...     )
    ...
    ...     # Execute with job parameters and environment variables
    ...     run_ds_job(
    ...         job=datastage_job,
    ...         job_parameters={'param1': 'value1', 'param2': 100},
    ...         env_variables={'ENV_VAR1': 'value1', 'ENV_VAR2': 'value2'}
    ...     )

.. skip: end

.. _preparing_data__pipelines__components__run_pipeline_job:

Run Pipeline Job
^^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.RUN_PIPELINE_JOB``

**Description**: Executes another pipeline job within the current pipeline. This allows you to create hierarchical pipelines where one pipeline can trigger another, enabling modular and reusable pipeline designs.

**Parameters**:

* ``job`` (required): The pipeline job to execute. Can be a Job object or a job reference string.

**Outputs**:

* Job run information and status

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> # Create a child pipeline first
    >>> @dsl.component
    ... def process_data(data: str) -> None:
    ...     print(f"Processing: {data}")
    >>>
    >>> @dsl.pipeline
    ... def child_pipeline() -> None:
    ...     process_data(data='test')
    >>>
    >>> child_pipeline_obj = project.create_pipeline(
    ...     name='Child Pipeline',
    ...     pipeline_function=child_pipeline
    ... )
    >>> child_job = project.create_job(name='Child Job', flow=child_pipeline_obj)
    >>>
    >>> # Create parent pipeline that runs the child
    >>> @dsl.pipeline
    ... def parent_pipeline() -> None:
    ...     run_pipeline = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_PIPELINE_JOB
    ...     )
    ...
    ...     # Execute the child pipeline job
    ...     run_pipeline(job=child_job)
    >>>
    >>> parent = project.create_pipeline(
    ...     name='Parent Pipeline',
    ...     pipeline_function=parent_pipeline
    ... )

.. skip: end

.. _preparing_data__pipelines__components__run_bash_script:

Run Bash Script
^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.RUN_BASH_SCRIPT``

**Description**: Executes a bash script within the pipeline. This component is useful for running shell commands, system operations, or integrating with command-line tools.

**Parameters**:

* ``script`` (required): The bash script to execute as a string.
* ``error_policy`` (optional): How to handle errors. Options: ``'fail_on_error'`` (default) or ``'continue_on_error'``.

**Outputs**:

* ``standard_output``: The stdout from the script execution
* ``standard_error``: The stderr from the script execution
* ``return_value``: The exit code from the script

**Example - Basic Usage**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def bash_pipeline() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...
    ...     # Execute a simple bash command
    ...     run_bash(script='echo 'Hello from pipeline!'')
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='Bash Script Pipeline',
    ...     pipeline_function=bash_pipeline
    ... )

.. skip: end

**Example - Capturing Output**:

.. skip: start

.. code-block:: python

    >>> @dsl.component
    ... def process_output(stdout: str) -> None:
    ...     print(f"Script output was: {stdout}")
    >>>
    >>> @dsl.pipeline
    ... def bash_with_output() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...
    ...     # Execute script and capture output
    ...     bash_task = run_bash(
    ...         script='echo 'Processing complete'',
    ...         error_policy='continue_on_error'
    ...     )
    ...
    ...     # Use the output in another component
    ...     process_output(stdout=bash_task.outputs['standard_output'])

.. skip: end

**Example - Error Handling**:

.. skip: start

.. code-block:: python

    >>> @dsl.pipeline
    ... def bash_with_error_handling() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...
    ...     # Continue even if script fails
    ...     bash_task = run_bash(
    ...         script='exit 42',
    ...         error_policy='continue_on_error'
    ...     )
    ...
    ...     # Check the return code
    ...     with dsl.If(bash_task.outputs['return_value'] != 0):
    ...         run_bash(script='echo 'Script failed!'')

.. skip: end

.. _preparing_data__pipelines__components__terminate_pipeline:

Terminate Pipeline
^^^^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.TERMINATE_PIPELINE``

**Description**: Terminates the pipeline execution. This component provides different modes for stopping the pipeline and handling running tasks.

**Parameters**:

* ``terminator_mode`` (required): How to terminate the pipeline. Options:

  * ``'stop'``: Stop the pipeline and cancel all running tasks immediately
  * ``'stop_wait'``: Stop the pipeline and wait for running tasks to complete
  * ``'abort'``: Abort the pipeline immediately without waiting
  * ``'abort_wait'``: Abort the pipeline but wait for running tasks to complete

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def pipeline_with_termination() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...     terminate = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.TERMINATE_PIPELINE
    ...     )
    ...
    ...     # Run a task
    ...     task1 = run_bash(script='echo 'Task 1'')
    ...
    ...     # Terminate the pipeline after task1
    ...     terminate_task = terminate(terminator_mode='stop_wait')
    ...     terminate_task.after(task1)

.. skip: end

.. _preparing_data__pipelines__components__terminate_loop:

Terminate Loop
^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.TERMINATE_LOOP``

**Description**: Breaks out of a loop iteration. This component allows you to exit a loop early based on conditions, similar to a ``break`` statement in programming.

**Parameters**:

* ``loop_status`` (required): The status to set when terminating the loop. Options:

  * ``'Completed'``: Mark the loop as completed successfully
  * ``'Failed'``: Mark the loop as failed

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def loop_with_break() -> None:
    ...     loop_component = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.LOOP_IN_PARALLEL
    ...     )
    ...     terminate_loop = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.TERMINATE_LOOP
    ...     )
    ...
    ...     # Loop through items
    ...     with loop_component(over=[1, 2, 3, 4, 5], parallelism=1):
    ...         # Break out of loop with completed status
    ...         terminate_loop(loop_status='Completed')

.. skip: end

.. _preparing_data__pipelines__components__loop_in_parallel:

Loop in Parallel
^^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.LOOP_IN_PARALLEL``

**Description**: Executes tasks in parallel for each item in a collection. This component creates a parallel loop where multiple iterations can run simultaneously, improving performance for independent tasks.

**Parameters**:

* ``over`` (required): The collection to iterate over (list of items)
* ``parallelism`` (optional): Maximum number of parallel executions (default: unlimited)

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.component
    ... def process_item(item: str) -> None:
    ...     print(f"Processing: {item}")
    >>>
    >>> @dsl.pipeline
    ... def parallel_loop_pipeline() -> None:
    ...     loop_component = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.LOOP_IN_PARALLEL
    ...     )
    ...
    ...     # Process items in parallel (up to 3 at a time)
    ...     with loop_component(over=['item1', 'item2', 'item3', 'item4'], parallelism=3) as item:
    ...         process_item(item=item)
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='Parallel Loop Pipeline',
    ...     pipeline_function=parallel_loop_pipeline
    ... )

.. skip: end

.. _preparing_data__pipelines__components__loop_in_sequence:

Loop in Sequence
^^^^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.LOOP_IN_SEQUENCE``

**Description**: Executes tasks sequentially for each item in a collection. Unlike parallel loops, this ensures items are processed one at a time in order.

**Parameters**:

* ``over`` (required): The collection to iterate over (list of items)

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.component
    ... def process_item_sequential(item: str) -> None:
    ...     print(f"Processing in order: {item}")
    >>>
    >>> @dsl.pipeline
    ... def sequential_loop_pipeline() -> None:
    ...     loop_component = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.LOOP_IN_SEQUENCE
    ...     )
    ...
    ...     # Process items one at a time in order
    ...     with loop_component(over=['first', 'second', 'third']) as item:
    ...         process_item_sequential(item=item)

.. skip: end

.. _preparing_data__pipelines__components__wait_for_file:

Wait for File
^^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.WAIT_FOR_FILE``

**Description**: Waits for a file to exist before proceeding with pipeline execution. This is useful for coordinating with external processes that create files.

**Parameters**:

* ``file`` (required): The file path to wait for
* ``wait_mode`` (required): How to wait for the file. Options:

  * ``'appear'``: Wait for the file to appear
  * ``'disappear'``: Wait for the file to disappear

* ``timeout_length`` (required): Maximum time to wait in format ``'HH:MM:SS'``

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def wait_for_file_pipeline() -> None:
    ...     wait_file = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.WAIT_FOR_FILE
    ...     )
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...
    ...     # Wait for a file to appear (timeout after 5 minutes)
    ...     wait_task = wait_file(
    ...         file='/path/to/expected/file.txt',
    ...         wait_mode='appear',
    ...         timeout_length='00:05:00'
    ...     )
    ...
    ...     # Process the file after it appears
    ...     process_task = run_bash(script='cat /path/to/expected/file.txt')
    ...     process_task.after(wait_task)

.. skip: end

.. _preparing_data__pipelines__components__wait_for_any:

Wait for Any
^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.WAIT_FOR_ANY``

**Description**: Waits for any one of the upstream tasks to complete before proceeding. This is useful when you have multiple parallel tasks and want to continue as soon as the first one finishes.

**Parameters**: None (uses ``.after()`` to specify upstream tasks)

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def wait_any_pipeline() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...     wait_any = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.WAIT_FOR_ANY
    ...     )
    ...
    ...     # Start two tasks with different durations
    ...     task1 = run_bash(script='sleep 5')
    ...     task2 = run_bash(script='sleep 10')
    ...
    ...     # Wait for any task to complete
    ...     wait_task = wait_any().after(task1, task2)
    ...
    ...     # Continue after the first task completes
    ...     run_bash(script='echo 'First task completed!'').after(wait_task)

.. skip: end

.. _preparing_data__pipelines__components__wait_for_all:

Wait for All
^^^^^^^^^^^^

**Component ID**: ``PipelineComponentId.WAIT_FOR_ALL``

**Description**: Waits for all upstream tasks to complete before proceeding. This ensures all parallel tasks finish before continuing with the next step.

**Parameters**: None (uses ``.after()`` to specify upstream tasks)

**Example**:

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>> from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    >>>
    >>> @dsl.pipeline
    ... def wait_all_pipeline() -> None:
    ...     run_bash = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.RUN_BASH_SCRIPT
    ...     )
    ...     wait_all = project.pipeline_components.get(
    ...         component_id=PipelineComponentId.WAIT_FOR_ALL
    ...     )
    ...
    ...     # Start multiple parallel tasks
    ...     task1 = run_bash(script='sleep 5')
    ...     task2 = run_bash(script='sleep 3')
    ...     task3 = run_bash(script='sleep 7')
    ...
    ...     # Wait for all tasks to complete
    ...     wait_task = wait_all().after(task1, task2, task3)
    ...
    ...     # Continue only after all tasks complete
    ...     run_bash(script='echo 'All tasks completed!'').after(wait_task)

.. skip: end

.. _preparing_data__pipelines__components__creating_custom:

Creating Custom Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create custom pipeline components for specialized functionality using the :py:meth:`Project.create_pipeline_component() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_pipeline_component>` method.

.. skip: start

.. code-block:: python

    >>> def process_data(input_text: str, multiplier: int = 2) -> str:
    ...     '''Process input text by repeating it.'''
    ...     print(f"Processing: {input_text}")
    ...     result = input_text * multiplier
    ...     print(f"Result: {result}")
    ...     return result
    >>>
    >>> # Create the custom component
    >>> custom_component = project.create_pipeline_component(
    ...     name='Process Data Component',
    ...     function=process_data,
    ...     description='Processes text by repeating it'
    ... )
    >>> custom_component
    PipelineComponent(component_id='...', name='Process Data Component', ...)

.. skip: end

Once created, you can use your custom component in pipelines just like built-in components:

.. skip: start

.. code-block:: python

    >>> @dsl.pipeline
    ... def my_pipeline() -> None:
    ...     # Get the custom component
    ...     processor = project.pipeline_components.get(
    ...         name='Process Data Component',
    ...         is_built_in=False
    ...     )
    ...
    ...     # Use it in the pipeline
    ...     result = processor(input_text='Hello', multiplier=3)
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='Pipeline with custom component',
    ...     pipeline_function=my_pipeline
    ... )

.. skip: end

Overwriting Custom Components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you need to update an existing custom component, you can use the ``overwrite`` parameter:

.. skip: start

.. code-block:: python

    >>> def improved_process_data(input_text: str, multiplier: int = 2) -> str:
    ...     '''Improved version of the data processor.'''
    ...     print(f"Processing (v2): {input_text}")
    ...     result = input_text * multiplier
    ...     return result.upper()
    >>>
    >>> # Overwrite the existing component
    >>> updated_component = project.create_pipeline_component(
    ...     name='Process Data Component',
    ...     function=improved_process_data,
    ...     description='Improved data processor',
    ...     overwrite=True
    ... )

.. skip: end

.. _preparing_data__pipelines__components__deleting_custom:

Deleting Custom Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can delete custom pipeline components (but not built-in ones) using the :py:meth:`Project.delete_pipeline_component() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_pipeline_component>` method.

.. skip: start

.. code-block:: python

    >>> # Get the custom component (using fixture)
    >>> project.delete_pipeline_component(custom_pipeline_component)
    <Response [204]>

.. skip: end

.. note::
    Built-in pipeline components cannot be deleted. Only custom components created by users can be removed.

.. warning::
    Deleting a component that is being used in existing pipelines may cause those pipelines to fail. Ensure the component is not in use before deleting it.
