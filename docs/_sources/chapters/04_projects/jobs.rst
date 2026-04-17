.. _projects__jobs:

Jobs
====

A job is an executable unit of work defined for a specific asset.
It acts as a reusable template that includes basic configuration and parameters.

The actual execution of a job is referred to as a job run.
A job run allows you to override the default configuration and captures runtime execution details.
The SDK provides functionality to interact with both jobs and job runs on the watsonx.data integration platform.

This includes operations such as:
    * Creating a job
    * Retrieving an existing job
    * Updating a job
    * Editing runtime settings of a batch job
    * Starting a job
    * Retrieving a job run
    * Refreshing a job run's state
    * Cancelling a job run
    * Retrieving logs from a job run
    * Deleting a job
    * Resetting an offset

Creating a Job
~~~~~~~~~~~~~~

In the UI, you can create and run a new job directly from the flow canvas by clicking the **Run** button.

.. image:: /_static/images/jobs/create_job.png
   :alt: Screenshot of the Job creation in the UI
   :align: center
   :width: 100%

|

To create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object within a
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` using the SDK,
first select the appropriate project from the :py:class:`~ibm_watsonx_data_integration.platform.Platform`,
then use the :py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>`
method to instantiate the job.

You must specify a reference to a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` object.
Additionally, you can provide optional configuration such as environment variables, schedules, retention policies, and runtime parameters.
The ``runtime_parameters`` parameter accepts either a dictionary or a :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` object.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object.

.. code-block:: python

    >>> # Create a job without runtime parameters
    >>> job = project.create_job(
    ...     flow=batch_flow,
    ...     name='Test Job',
    ...     description='...',
    ... )
    >>> job
    Job(name='Test Job', ...)

You can override parameter set values at job creation time by passing a dictionary to ``runtime_parameters``.
The dictionary keys should be in the format ``"parameter_set_name.parameter_name"`` for external parameters,
or just ``"parameter_name"`` for local parameters:

.. code-block:: python

    >>> # Create a job with runtime parameters to override parameter set values
    >>> job_with_runtime_parameters = project.create_job(
    ...     flow=batch_flow_with_parameters,
    ...     name='Test Job with Parameters',
    ...     description='Job with custom parameter values',
    ...     runtime_parameters={
    ...         'myparamset.param1': 'production_db',
    ...         'localparam1': 'custom_value'
    ...     }
    ... )
    >>> job_with_runtime_parameters
    Job(name='Test Job with Parameters', ...)


Retrieving an Existing Job
~~~~~~~~~~~~~~~~~~~~~~~~~~

To list existing jobs in the UI, navigate to the **Jobs** tab in project view.

.. image:: /_static/images/jobs/list_jobs.png
   :alt: Screenshot of listing jobs
   :align: center
   :width: 100%

|

In the SDK, jobs can be retrieved using the :py:class:`Project.jobs <ibm_watsonx_data_integration.cpd_models.project_model.Project.jobs>` property.
You can also filter the returned jobs based on attributes including
``space_id``, ``job_id``, ``job_type``, and ``run_id``.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Jobs` object.

.. code-block:: python

    >>> # Returns all jobs
    >>> project.jobs
    [Job(...), ...]
    >>> # Returns the first job matching the given `name`
    >>> project.jobs.get(name='Test Job')
    Job(name='Test Job', ...)
    >>> # Returns a list of all jobs that match the given `name`
    >>> project.jobs.get_all(name='Test Job')
    [Job(name='Test Job', ...)]

.. _projects__jobs__updating_a_job:

Updating a Job
~~~~~~~~~~~~~~

In the UI, you can update a job by selecting its title from the jobs list.
To update a job, click the pencil icon in the top bar.

.. image:: /_static/images/jobs/job_details_page_update_job.jpeg
   :alt: Screenshot of the Job Details page with the pencil icon highlighted
   :align: center
   :width: 100%

|

In the SDK, updating a job is possible via your :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` instance.
First, modify the properties of the existing job, then update it using the
:py:meth:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.update_job` method.
You can modify properties and ``configuration`` fields of the :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object.
This method returns an HTTP response indicating the status of the update operation.
The response also includes the updated job definition.

.. code-block:: python

    >>> job.name = 'New name'
    >>> job.description = 'New description.'
    >>> project.update_job(job)
    <Response [200]>

You can display the properties of a job by calling the :py:meth:`Job.print_json() <ibm_watsonx_data_integration.cpd_models.job_model.Job.print_json>` method.

.. code-block:: python

    >>> job.print_json()
    {
        "metadata": {
            "name": "New name",
            "description": "New description.",
            ...
        },
        ...
        "configuration": {...},
        ...
    }


.. note::
    Most fields other than metadata and configuration are runtime settings for batch jobs. These settings can be set only by a dedicated method. This is explained in the next section.

.. _projects__jobs__editing_a_batch_job:

Editing runtime settings (Batch)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Batch flow jobs have runtime settings you can edit per job.
In the UI, you can change runtime settings in the same location where you :ref:`update a job<projects__jobs__updating_a_job>`.
Batch flows include additional settings.

.. image:: /_static/images/jobs/job_details_page_update_job.jpeg
   :alt: Screenshot of the edit configurations button
   :align: center
   :width: 100%

|

.. image:: /_static/images/jobs/runtime_settings.png
   :alt: Screenshot of the Runtime Settings page
   :align: center
   :width: 100%

|

In the SDK, you can call the :py:meth:`~ibm_watsonx_data_integration.cpd_models.job_model.Job.edit_configuration` method on the job instance. The following settings can be changed:

* ``environment``: The internal name of the batch environment to use. To find the internal name of a batch environment, you can either list all internal names with :py:meth:`Project.list_batch_environments() <ibm_watsonx_data_integration.cpd_models.project_model.Project.list_batch_environments>` or call :py:meth:`Project.get_batch_environment() <ibm_watsonx_data_integration.cpd_models.project_model.Project.get_batch_environment>` with the display name.


.. code-block:: python

    >>> project.list_batch_environments()
    ['default_datastage_px', 'default_datastage_px_large', 'default_datastage_px_medium']
    >>> project.get_batch_environment('Default DataStage PX S')
    'default_datastage_px'


* ``warn_limit``: The number of warnings allowed before the stages are stopped. Takes an ``int`` greater than 0 or ``None`` for no limit.
* ``retention_days``: The number of days to keep a job run. Cannot be set if ``retention_amount`` is also set. Takes an ``int`` greater than 0 or ``None`` for no limit.
* ``retention_amount``: The number of job runs to keep in total. Cannot be set if ``retention_days`` is also set. Takes an ``int`` greater than 0 or ``None`` for no limit.
* ``parameter_value_sets``: The value sets to use in this job. Takes a ``list`` of ``tuples`` where the first value is the name of the parameter set and the second value is the value set to use for that parameter set.
* ``job_parameters``: The values to use for the external and local parameters in this job. Takes a ``list`` of ``tuples`` where the first value is the name of the parameter and the second value is the value to use for that parameter. For external parameters, make sure the name of the parameter set comes before the name of the parameter: ``parameter_set.parameter``.
* ``schedule``: The schedule on which to run the job. Takes a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Schedule` object or ``None`` for no schedule.
* ``notify_success``: Takes a ``boolean`` indicating whether to notify on success.
* ``notify_warning``: Takes a ``boolean`` indicating whether to notify on warning.
* ``notify_failure``: Takes a ``boolean`` indicating whether to notify on failure.

.. code-block:: python

    >>> import datetime
    >>> from ibm_watsonx_data_integration.cpd_models.job_model import Schedule
    >>> schedule = Schedule(start_date = datetime.datetime(2030, 12, 11, 22, 17), repeat_mode = 'daily', repeat_value = datetime.time(23, 17))
    >>> batch_job.edit_configuration(
    ...    environment='default_datastage_px',
    ...    retention_amount=100,
    ...    parameter_value_sets=[('myparamset', 'value_set_1'), ('paramset2', 'valset_2')],
    ...    job_parameters=[('myparamset.param1', 'myvalue'), ('localparam1', 'myvalue2')],
    ...    schedule=schedule,
    ...    notify_success=True,
    ...    notify_warning=False,
    ...    notify_failure=True
    ... )
    <Response [200]>


Starting a Job
~~~~~~~~~~~~~~

In the UI, you can start a job from the **Job Details** page by clicking the play icon.

.. image:: /_static/images/jobs/job_details_page_start_job.jpeg
   :alt: Screenshot of the Job Details page with the play icon highlighted
   :align: center
   :width: 100%

|

A job instance serves as a template to actually execute the flow for which the job was created.
Call the :py:meth:`Job.start() <ibm_watsonx_data_integration.cpd_models.job_model.Job.start>` method on the job instance.
The ``name`` and ``description`` parameters are optional; if not provided, default values will be used (``name='job run'``, ``description=''``).
Additionally, you can further configure the job run by passing the ``configuration`` and ``runtime_parameters`` parameters to this method.
The ``runtime_parameters`` parameter accepts either a dictionary or a :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` object.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.JobRun` object.

.. code-block:: python

    >>> # Start a job run without runtime parameters
    >>> job_run = job.start(name='Test Job Run', description='...')
    >>> job_run
    JobRun(...job_name='New name'...)

You can override parameter values for a specific job run by passing a dictionary to ``runtime_parameters``.
This allows you to run the same job with different parameter values without modifying the job itself.
The dictionary keys should be in the format ``"parameter_set_name.parameter_name"`` for external parameters,
or just ``"parameter_name"`` for local parameters:

.. code-block:: python

    >>> # Start a job run with runtime parameters to override values
    >>> job_run_with_runtime_parameters = job_with_runtime_parameters.start(
    ...     name='Test Job Run with Custom Parameters',
    ...     description='Run with overridden parameter values',
    ...     runtime_parameters={
    ...         'myparamset.param1': "value",
    ...         'localparam1': 'override_value'
    ...     }
    ... )
    >>> job_run_with_runtime_parameters
    JobRun(...job_name='Test Job with Parameters'...)

Retrieving a Job Run
~~~~~~~~~~~~~~~~~~~~

To view job runs in the UI, go to the **Jobs** tab, select a job, and open the **Job Details** page.

.. image:: /_static/images/jobs/list_job_runs.png
   :alt: Screenshot of Job Runs list
   :align: center
   :width: 100%

|

In the SDK, job runs can be retrieved using the :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job.job_runs` property.
You can filter the returned jobs by ``space_id`` and ``state`` attributes.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.JobRuns` object.

.. code-block:: python

    >>> # Returns a list of all job runs for given job
    >>> job.job_runs
    [JobRun(...job_name='New name'...)]
    >>> # Returns a list of all job runs
    >>> job.job_runs.get_all()
    [JobRun(...job_name='New name'...)]


Refreshing a Job Run State
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can refresh a job's status by clicking the three dots next to a job and then selecting **Refresh status**.

.. image:: /_static/images/jobs/job_run_refresh_status.png
   :alt: Screenshot of the refresh job status button
   :align: center
   :width: 100%

To refresh a job run using the SDK, call the :py:meth:`JobRun.refresh_status() <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.refresh_status>` method.
It updates the :py:attr:`JobRun.state <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.state>` property.

.. code-block:: python

    >>> job_run.refresh_status()
    <Response [200]>
    >>> job_run.state
    <JobRunState...>


Cancelling a Job Run
~~~~~~~~~~~~~~~~~~~~

In the UI, you can cancel a job run by selecting **Cancel** from the three-dot menu of the selected run.

.. image:: /_static/images/jobs/cancel_job_run.png
   :alt: Screenshot of canceling Job Run
   :align: center
   :width: 100%

|

To cancel a running job run using the SDK, use the :py:meth:`JobRun.cancel() <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.cancel>`
method on the job run instance.
This method returns an HTTP response indicating the status of the operation.

.. skip: start 'cant determine job state'

.. code-block:: python

    >>> job_run.cancel()
    <Response [204]>

.. skip: end

.. invisible-code-block: python

    >>> from tests.integration.test_job import wait_for_expected_state
    >>> job_run.refresh_status()
    <Response [200]>
    >>> _ = job_run.cancel() if job_run.state in job_working_states else None
    >>> _ = wait_for_expected_state(job, job_run, job_finished_states) if job_run.state in job_working_states else None
    >>> job_run_with_runtime_parameters.refresh_status()
    <Response [200]>
    >>> _ = job_run_with_runtime_parameters.cancel() if job_run_with_runtime_parameters.state in job_working_states else None
    >>> _ = wait_for_expected_state(job, job_run_with_runtime_parameters, job_finished_states) if job_run_with_runtime_parameters.state in job_working_states else None
    >>> project.delete_job(job_with_runtime_parameters)
    <Response [204]>


Retrieving Job Run Logs
~~~~~~~~~~~~~~~~~~~~~~~

.. TODO Add UI screenshot when logs will be visible in UI

Runtime logs for a job run execution are stored in the
:py:class:`JobRun.logs <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.logs>` property.
It returns a list where each entry is a string containing a log message.

.. skip: start 'not supported for streaming flows'

.. code-block:: python

    >>> job_run.logs
    [
        '##I IIS-DSEE-TOSH-00397 2025-05-27 14:27:27(000) Starting job Job for Test flow 1750236869686',
        '##I IIS-DSEE-TOSH-00408 2025-05-27 14:27:27(000) Job Parameters:',
        ...
    ]

.. skip: end

Retrieving Job Run metrics (Batch)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Batch job runs provide detailed performance metrics that can be accessed through the
:py:attr:`JobRun.metrics <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.metrics>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.job_metrics.BatchJobRunMetrics` object
containing link and stage metrics for the job run.

The metrics include:

* **Link Metrics**: Information about data flow between stages, including rows read/written, timing, and state
* **Stage Metrics**: Performance data for each stage, including CPU time, memory usage, partition counts, and row counts

.. skip: start 'job_run reference streaming run not batch, need to adjust'

.. code-block:: python

    >>> # Get metrics for a completed batch job run
    >>> metrics = job_run.metrics
    >>> metrics
    BatchJobRunMetrics(link_metrics=[...], stage_metrics=[...])

**Accessing Link Metrics**

Link metrics provide information about data transfer between stages:

.. code-block:: python

    >>> # View all link metrics
    >>> metrics.link_metrics
    [
        {
            "rows_read": 1000,
            "rows_written": 1000,
            "source": "Sequential_File_1",
            "dest": "Transformer_1",
            "link_name": "DSLink3",
            "state": "finished"
        },
        ...
    ]

    >>> # Access specific link metrics by index
    >>> link = metrics.link_metrics[0]
    >>> link.rows_read
    1000
    >>> link.source
    'Sequential_File_1'
    >>> link.dest
    'Transformer_1'

**Accessing Stage Metrics**

Stage metrics provide performance information for each stage in the flow:

.. code-block:: python

    >>> # View all stage metrics
    >>> metrics.stage_metrics
    [
        {
            "rows_read": 1000,
            "rows_written": 1000,
            "stage_type": "PxSequentialFile",
            "stage_name": "Sequential_File_1",
            "num_partitions": 4,
            "state": "finished"
        },
        ...
    ]

    >>> # Access specific stage metrics by index
    >>> stage = metrics.stage_metrics[0]
    >>> stage.rows_read
    1000
    >>> stage.stage_name
    'Sequential_File_1'
    >>> stage.num_partitions
    4

**Filtering Metrics**

The metrics lists support filtering using the ``get()`` and ``get_all()`` methods:

.. code-block:: python

    >>> # Get first link from a specific source stage
    >>> link = metrics.link_metrics.get(source='Sequential_File_1')
    >>> link.rows_read
    1000

    >>> # Get all links to a specific destination stage
    >>> links = metrics.link_metrics.get_all(dest='Transformer_1')
    >>> len(links)
    2

    >>> # Get first stage of a specific type
    >>> stage = metrics.stage_metrics.get(stage_type='PxSequentialFile')
    >>> stage.stage_name
    'Sequential_File_1'

    >>> # Get all stages in finished state
    >>> finished_stages = metrics.stage_metrics.get_all(state='finished')
    >>> len(finished_stages)
    5

**Refreshing Metrics**

You can refresh the metrics of a job run by calling :py:meth:`JobRun.refresh_metrics() <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.refresh_metrics>` to get the latest metrics.

.. code-block:: python

    >>> job_run.refresh_metrics()
    <Response [200]>

.. skip: end

Deleting a Job
~~~~~~~~~~~~~~

In the UI, you can delete a job by selecting its title from the jobs list.
To delete a job, click the trash icon in the top bar.

.. image:: /_static/images/jobs/job_details_page_delete_job.jpeg
   :alt: Screenshot of the Job Details page with the trash icon highlighted
   :align: center
   :width: 100%

|

In the SDK, to delete a job you can pass the job instance to the :py:meth:`Project.delete_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_job>` method.
This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> project.delete_job(job)
    <Response [204]>

Resetting an Offset (Streaming)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. warning::
   This method doesn't work on batch jobs.

Jobs use offsets to track the last processed data before they were completed or cancelled.
In certain cases, it may be necessary to reset a job’s offset to reprocess data.
To reset the job offset in the UI, select **Restart from initial offset** from the edit job screen.

.. image:: /_static/images/jobs/reset_job_offsets.jpeg
    :alt: Screenshot of resetting Job offsets
    :align: center
    :width: 100%

|

Job offsets can be reset using the :py:meth:`Job.reset_offset() <ibm_watsonx_data_integration.cpd_models.job_model.Job.reset_offset>`
method on the job instance.

.. skip: start 'reset_offset not supported for data_intg_flow job type'

.. code-block:: python

    >>> job.reset_offset()
    <Response [200]>

.. skip: end
