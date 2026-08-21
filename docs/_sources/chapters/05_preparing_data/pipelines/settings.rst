.. _preparing_data__pipelines__settings:

Pipeline Settings
=================

This section covers project-level settings that affect how pipelines are executed.

.. _preparing_data__pipelines__settings__optimized_runner:

DataStage Optimized Pipeline Runner
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The DataStage optimized pipeline runner compiles pipelines into Python code and runs them directly inside the engine runtime,
eliminating the REST API call overhead that the standard runner incurs for every nested pipeline invocation.
When used with the :ref:`Run Pipeline Job <preparing_data__pipelines__components__run_pipeline_job>` component,
nested pipeline jobs execute inline instead of being dispatched as separate API calls —
most beneficial in workflows with deeply nested or many sequential pipeline stages.

.. note::
    This setting applies to all pipelines in the project.
    To use a different runner for a specific workflow, run the pipeline from a separate project where the setting is turned off.

.. _preparing_data__pipelines__settings__optimized_runner__enabling:

Enabling the Optimized Runner
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::
    The instructions below apply to **On-Premises** environments only.
    For **SaaS**, this option is available only for projects with a remote engine for batch enabled.

You can enable the optimized pipeline runner from project-level settings in the UI:

#. Create or open an existing project.
#. In the **Manage** tab, choose the **Pipeline** section from the left-side options.
#. Go to the **DataStage optimized pipeline runner** section.
#. Select the **Run using optimized pipeline runner** option and click **Save**.

.. image:: /_static/images/pipelines/datastage_optimized_runner_on_prem.png
   :alt: Screenshot of the DataStage settings (On-Premises)
   :align: center
   :width: 100%

|

Once enabled, all new pipelines in the project run using the optimized runner. Open an existing pipeline from **Assets → Flows → Pipelines** and click **Run pipeline** to execute it with the optimized runner.

.. _preparing_data__pipelines__settings__optimized_runner__sdk:

Working with the Optimized Runner via the SDK
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can read and update the optimized pipeline runner setting programmatically using the
:py:attr:`Project.pipeline_settings <ibm_watsonx_data_integration.cpd_models.project_model.Project.pipeline_settings>` property.

.. _preparing_data__pipelines__settings__optimized_runner__sdk__reading:

Reading the Current Setting
'''''''''''''''''''''''''''

.. skip: start 'Skipped to avoid modifying project settings during documentation tests.'

.. code-block:: python

    >>> project.pipeline_settings.runner_type
    <PipelineRunnerType.PIPELINE_RUNNER: 'PIPELINE_RUNNER'>

.. skip: end

.. _preparing_data__pipelines__settings__optimized_runner__sdk__enabling:

Enabling the Optimized Runner
'''''''''''''''''''''''''''''

.. skip: start 'Skipped to avoid modifying project settings during documentation tests.'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.services.pipelines.models.pipeline_settings_model import PipelineRunnerType
    >>>
    >>> project.pipeline_settings.runner_type = PipelineRunnerType.DATASTAGE_OPTIMIZED_RUNNER
    >>>
    >>> response = platform.update_project(project)
    >>> response.status_code
    200
    >>> project.pipeline_settings.runner_type
    <PipelineRunnerType.DATASTAGE_OPTIMIZED_RUNNER: 'DATASTAGE_OPTIMIZED_RUNNER'>

.. skip: end

.. _preparing_data__pipelines__settings__optimized_runner__sdk__disabling:

Disabling the Optimized Runner
''''''''''''''''''''''''''''''

To revert to the standard pipeline runner, set the runner type back to
:py:attr:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_settings_model.PipelineRunnerType.PIPELINE_RUNNER`.

.. skip: start 'Skipped to avoid modifying project settings during documentation tests.'

.. code-block:: python

    >>> project.pipeline_settings.runner_type = PipelineRunnerType.PIPELINE_RUNNER
    >>>
    >>> response = platform.update_project(project)
    >>> response.status_code
    200

.. skip: end

.. _preparing_data__pipelines__settings__optimized_runner__compile_status:

Tracking Compile Status
^^^^^^^^^^^^^^^^^^^^^^^

When the optimized runner is enabled, each pipeline must be compiled before it can run.

.. TODO: SDK support for reading pipeline compile status is not yet implemented. Add code example and status table once available.
