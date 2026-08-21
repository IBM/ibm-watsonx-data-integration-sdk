.. _preparing_data__pipelines__examples:

Pipeline Examples
=================

This section provides practical examples demonstrating key pipeline capabilities. Each example focuses on a single topic so you can see exactly how each feature works and combine them in your own workflows.

.. contents:: In this section
   :local:
   :depth: 1

.. _preparing_data__pipelines__examples__cel_expressions:

CEL Expressions
~~~~~~~~~~~~~~~

CEL (*Common Expression Language*) expressions let you perform runtime computations inside a pipeline — arithmetic, comparisons, string operations, and more — without writing a custom Python component.

Use :py:class:`~ibm_watsonx_data_integration.services.pipelines.cel.CEL` to build expressions:
:py:meth:`~ibm_watsonx_data_integration.services.pipelines.cel.CEL.literal` for fixed values you already know (a string prefix, a constant number),
and :py:meth:`~ibm_watsonx_data_integration.services.pipelines.cel.CEL.expr` for values that come from user variables or task outputs at runtime.


.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.cel import CEL
    from ibm_watsonx_data_integration.services.pipelines.models import UserVariable
    from kfp import dsl


    # Access a user variable at runtime
    table_var = UserVariable(name='table_name', default_value='orders')
    table_expr = CEL.expr('vars.table_name', str)

    # Use CEL.expr to build an expression from a user variable
    retry_var = UserVariable(name='retry_count', default_value=0)
    retry_exceeded = CEL.expr('vars.retry_count', int) >= 3

    # Use CEL.literal for a static value combined with a user variable
    prefixed_table = (CEL.literal('processing_') + CEL.expr('vars.table_name', str)).evaluate()

.. note::
    Always call ``.evaluate()`` on a CEL expression before passing it to a component or a ``dsl.If`` condition.


For the full list of avialabe methods see the :py:class:`~ibm_watsonx_data_integration.services.pipelines.cel.CELExpression` API reference.


.. _preparing_data__pipelines__examples__user_variables:

User Variables via Set User Variable Component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:py:class:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_component_model.UserVariable`
objects hold pipeline-scoped state that persists across stages. They must be:

1. Declared **outside** the ``@dsl.pipeline`` function.
2. Passed to :py:meth:`Project.create_pipeline() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_pipeline>`
   via the ``user_variables`` list.
3. Updated inside the pipeline using the ``set_user_variable`` component.
4. Read inside another component using :py:meth:`~ibm_watsonx_data_integration.services.pipelines.cel.CEL.expr`
   with the ``vars.<name>`` path.

**Declaring and setting user variables**

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.cel import CEL
    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId, UserVariable
    from kfp import dsl


    run_bash = project.pipeline_components.get(component_id=PipelineComponentId.RUN_BASH_SCRIPT)
    run_datastage_job = project.pipeline_components.get(component_id=PipelineComponentId.RUN_DATASTAGE_JOB)
    set_user_variables = project.pipeline_components.get(component_id=PipelineComponentId.SET_USER_VARIABLE)

    # Declare variables, name must be a unique string identifier
    table_var = UserVariable(name='table_name', default_value='orders')
    counter_var = UserVariable(name='run_counter', default_value=0, pipeline_result=True)

    ds_job = project.jobs.get(name='Load Table Job')


    @dsl.pipeline
    def pipeline_with_user_vars() -> None:
        # Set the table name to process
        set_user_variables(
            variables={table_var: 'customers', counter_var: 1},
            outputs={},
        )

        # Read the variable and pass it to the DataStage job
        run_datastage_job(
            job=ds_job,
            job_parameters={
                'TABLE_NAME': CEL.expr('vars.table_name', str).evaluate(),
            },
        )

    project.create_pipeline(
        name='Pipeline with User Variables',
        pipeline_function=pipeline_with_user_vars,
        overwrite=True,
        user_variables=[table_var, counter_var],
    )

.. skip: end

.. note::
    Variables marked with ``pipeline_result=True`` are surfaced as pipeline outputs
    and can be inspected after the job run completes.


.. _preparing_data__pipelines__examples__if_else:

If / Else Conditions
~~~~~~~~~~~~~~~~~~~~

Use the Kubeflow ``dsl.If`` / ``dsl.Else`` context managers to branch pipeline execution based
on the value of a task output or a CEL expression.

**Branching on a bash script exit code**

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    from kfp import dsl


    run_bash = project.pipeline_components.get(component_id=PipelineComponentId.RUN_BASH_SCRIPT)


    @dsl.pipeline
    def branch_on_exit_code() -> None:
        # Run a validation script (allow it to fail without stopping the pipeline)
        validate = run_bash(
            script='test -f /data/input.csv && echo \'ok\' || exit 1',
            error_policy='continue_on_error',
        )

        # Branch based on the exit code
        with dsl.If(condition=validate.outputs['return_value'] == 0):
            run_bash(script='echo \'File found — starting pipeline.\'')
        with dsl.Else():
            run_bash(script='echo \'File missing — skipping pipeline.\'')

    project.create_pipeline(
        name='Branch on Exit Code',
        pipeline_function=branch_on_exit_code,
        overwrite=True,
    )

.. skip: end

**Branching on a CEL expression derived from a user variable**

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.cel import CEL
    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId, UserVariable
    from kfp import dsl


    run_bash = project.pipeline_components.get(component_id=PipelineComponentId.RUN_BASH_SCRIPT)
    set_user_variables = project.pipeline_components.get(component_id=PipelineComponentId.SET_USER_VARIABLE)
    terminate_pipeline = project.pipeline_components.get(component_id=PipelineComponentId.TERMINATE_PIPELINE)

    # Declare a user variable that controls which environment to target
    environment_var = UserVariable(name='environment', default_value='dev')


    @dsl.pipeline
    def branch_on_environment() -> None:
        set_user_variables(variables={environment_var: 'prod'}, outputs={})

        # Run different jobs per environment
        env_expr = CEL.expr('vars.environment', str)

        with dsl.If(condition=env_expr == 'prod'):
            run_bash(script='echo \'Running production pipeline\'')
        with dsl.Else():
            run_bash(script='echo \'Running development pipeline\'')

    project.create_pipeline(
        name='Branch on Environment',
        pipeline_function=branch_on_environment,
        overwrite=True,
        user_variables=[environment_var],
    )

.. skip: end


.. _preparing_data__pipelines__examples__env_variables_bash:

Environment Variables in Bash Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Both the :ref:`Run Bash Script <preparing_data__pipelines__components__run_bash_script>` and
:ref:`Run DataStage Job <preparing_data__pipelines__components__run_datastage_job>` components
accept an ``env_variables`` dictionary. Values are available as standard shell environment
variables (``$VAR_NAME``) inside the script body.

**Passing static environment variables**

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId
    from kfp import dsl


    run_bash = project.pipeline_components.get(component_id=PipelineComponentId.RUN_BASH_SCRIPT)


    @dsl.pipeline
    def bash_with_env_vars() -> None:
        script = (
            'echo \'${ENV_VAR_SET_BY_SDK}\'\n'
            'echo \'${INT_VAR}\'\n'
            'echo \'${FLOAT_VAR}\''
        )
        run_bash(
            script=script,
            env_variables={
                'ENV_VAR_SET_BY_SDK': 'test string value',
                'INT_VAR': 10,
                'FLOAT_VAR': 67.56,
            },
        )


    project.create_pipeline(
        name='Bash with Env Vars',
        pipeline_function=bash_with_env_vars,
        overwrite=True,
    )

.. skip: end

**Passing a dynamic value from a user variable via CEL**

Use :py:meth:`~ibm_watsonx_data_integration.services.pipelines.cel.CEL.expr` to read a
user variable at runtime and forward it as an environment variable.

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.cel import CEL
    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId, UserVariable
    from kfp import dsl


    run_bash = project.pipeline_components.get(component_id=PipelineComponentId.RUN_BASH_SCRIPT)
    set_user_variables = project.pipeline_components.get(component_id=PipelineComponentId.SET_USER_VARIABLE)

    region_var = UserVariable(name='target_region', default_value='us-south')


    @dsl.pipeline
    def bash_with_dynamic_env() -> None:
        # Update the variable earlier in the pipeline if needed
        set_user_variables(variables={region_var: 'eu-de'}, outputs={})

        run_bash(
            script='echo \'Deploying to ${TARGET_REGION}\'',
            env_variables={
                'TARGET_REGION': CEL.expr('vars.target_region', str).evaluate(),
            },
        )

    project.create_pipeline(
        name='Bash with Dynamic Env',
        pipeline_function=bash_with_dynamic_env,
        overwrite=True,
        user_variables=[region_var],
    )

.. skip: end

**Passing environment variables to a DataStage job**

.. skip: start

.. code-block:: python

    from ibm_watsonx_data_integration.services.pipelines.cel import CEL
    from ibm_watsonx_data_integration.services.pipelines.models import PipelineComponentId, UserVariable
    from kfp import dsl


    run_datastage_job = project.pipeline_components.get(component_id=PipelineComponentId.RUN_DATASTAGE_JOB)
    set_user_variables = project.pipeline_components.get(component_id=PipelineComponentId.SET_USER_VARIABLE)

    schema_var = UserVariable(name='db_schema', default_value='PUBLIC')
    ds_job = project.jobs.get(name='Load Orders Job')


    @dsl.pipeline
    def datastage_with_env() -> None:
        set_user_variables(variables={schema_var: 'SALES'}, outputs={})

        run_datastage_job(
            job=ds_job,
            env_variables={
                'DB_SCHEMA': CEL.expr('vars.db_schema', str).evaluate(),
                'BATCH_DATE': '2024-01-15',
            },
        )

    project.create_pipeline(
        name='DataStage with Env Variables',
        pipeline_function=datastage_with_env,
        overwrite=True,
        user_variables=[schema_var],
    )

.. skip: end
