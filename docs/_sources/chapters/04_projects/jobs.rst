.. _projects__jobs:

Jobs
====
|

A job is an executable unit of work defined for a specific asset.
It acts as a reusable template that includes basic configuration and parameters.

The actual execution of a job is referred to as a job run.
A job run allows you to override the default configuration and captures runtime execution details.
The SDK provides functionality to interact with both jobs and job runs on the watsonx.data integration platform.

This includes operations such as:
    * Creating a job
    * Updating a job
    * Starting a job
    * Cancelling a job run
    * Retrieving logs from a job run
    * Deleting a job

Creating a Job
~~~~~~~~~~~~~~

In the UI, you can create and run a new Job directly from flow canvas by clicking **Run** button.

.. image:: ../../_static/images/jobs/create_job.png
   :alt: Screenshot of the Job creation in the UI
   :align: center
   :width: 100%

|

To create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object within a
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` using SDK,
first select the appropriate project from the :py:class:`~ibm_watsonx_data_integration.platform.Platform`,
then use the :py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>`
method to instantiate the job.

You must specify a reference to :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` object.
Additionally, you can provide optional configuration such as environment variables or job parameters.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object.

.. code-block:: python

    >>> job = project.create_job(
    ...     flow=flow,
    ...     name='Test Job',
    ...     description='...',
    ... )
    >>> job
    Job(name='Test Job', ...)


Retrieving an Existing Job
~~~~~~~~~~~~~~~~~~~~~~~~~~

To list existing jobs in the UI, navigate to **Jobs** tab in project view.

.. image:: ../../_static/images/jobs/list_jobs.png
   :alt: Screenshot of listing jobs
   :align: center
   :width: 100%

|

Jobs can be retrieved using :py:class:`Project.jobs <ibm_watsonx_data_integration.cpd_models.project_model.Project.jobs>` property.
You can also further filter and refine the jobs returned based on attributes including
``space_id``, ``job_id``, ``job_type`` and ``run_id``.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Jobs` object.

.. code-block:: python

    >>> # Returns all jobs
    >>> project.jobs
    [Job(name='Test Job', ...)]
    >>> # Returns first job matching given `name`
    >>> project.jobs.get(name='Test Job')
    Job(name='Test Job', ...)
    >>> # Return a list of all jobs that match given `name`
    >>> project.jobs.get_all(name='Test Job')
    [Job(name='Test Job', ...)]

Updating a Job
~~~~~~~~~~~~~~

In the UI, you can update a job by selecting its title from the jobs list.
To update a job, click the pencil icon in the top bar.

.. image:: ../../_static/images/jobs/job_details_page_update_job.jpeg
   :alt: Screenshot of the Job Details page with the pencil icon highlighted
   :align: center
   :width: 100%

|

Updating a job is also possible via :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` instance.
First, modify the properties of the existing job, then update it using the
:py:meth:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.update_job` method.
This method returns an HTTP response indicating the status of the update operation.
The response also includes the updated job definition.

.. code-block:: python

    >>> job.metadata.name = 'New name'
    >>> job.metadata.description = 'New description.'
    >>> project.update_job(job)
    <Response [200]>

Starting a Job
~~~~~~~~~~~~~~

In the UI, you can start job from **Job Details** page by clicking play icon.

.. image:: ../../_static/images/jobs/job_details_page_start_job.jpeg
   :alt: Screenshot of the Job Details page with the play icon highlighted
   :align: center
   :width: 100%

|

A job instance serves as a template to actually execute the flow for which the job was created.
Call :py:meth:`~ibm_watsonx_data_integration.cpd_models.job_model.Job.start` method on the job instance.
You can pass the ``name`` and ``description`` parameters to define the job run.
Additionally, you can further configure the job run by passing the ``configuration``,
``job_parameters`` and ``parameter_sets`` parameters to this method.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.JobRun` object.

.. code-block:: python

    >>> job_run = job.start(name='Test Job Run', description='...')
    >>> job_run
    JobRun(name='job run', ..., state='Queued')

Retrieving a Job Run
~~~~~~~~~~~~~~~~~~~~

In the UI, list the job runs by navigating to the **Jobs** tab, selecting a job, and viewing the **Job Details** page.

.. image:: ../../_static/images/jobs/list_job_runs.png
   :alt: Screenshot of Job Runs list
   :align: center
   :width: 100%

|

Job Runs can be retrieved using :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job.job_runs` property.
You can also further filter and refine the jobs returned based on attributes including
``space_id`` and ``states``.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.JobRuns` object.

.. code-block:: python

    >>> # Returns a list of all job runs for given job
    >>> job.job_runs
    [JobRun(name='job run', ..., state='Queued')]
    >>> # Returns a list of all job runs which status is `Running`
    >>> from ibm_watsonx_data_integration.cpd_models.job_model import JobRunState
    >>> job.job_runs.get_all(states=[JobRunState.Queued])
    [JobRun(name='job run', ..., state='Queued')]

.. invisible-code-block: python

    >>> from tests.integration.test_job import wait_for_expected_state
    >>> wait_for_expected_state(job, job_run, JobRunState.Running)


Cancelling a Job Run
~~~~~~~~~~~~~~~~~~~~

In the UI, you can cancel a job run by selecting **Cancel** from the three-dot menu of the selected run.

.. image:: ../../_static/images/jobs/cancel_job_run.png
   :alt: Screenshot of canceling Job Run
   :align: center
   :width: 100%

|

To cancel a running job run using SDK, use the :py:meth:`~ibm_watsonx_data_integration.cpd_models.job_model.JobRun.cancel`
method on the job run instance.
This method returns an HTTP response indicating the status of the operation.

.. code-block:: python

    >>> job_run.cancel()
    <Response [204]>

.. invisible-code-block: python

    >>> from tests.integration.test_job import wait_for_expected_state
    >>> wait_for_expected_state(job, job_run, JobRunState.Canceled)

Retrieving a Job Run logs
~~~~~~~~~~~~~~~~~~~~~~~~~

.. TODO Add UI screenshot when logs will be visible in UI

Runtime logs for a job run execution are stored in the
:py:class:`JobRun.logs <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.logs>` property.
It returns a list where each entry is a string containing a log message.

.. skip: start "not supported for streamsets flows"

.. code-block:: python

    >>> job_run.logs
    [
        '##I IIS-DSEE-TOSH-00397 2025-05-27 14:27:27(000) Starting job Job for Test flow 1750236869686',
        '##I IIS-DSEE-TOSH-00408 2025-05-27 14:27:27(000) Job Parameters:',
        ...
    ]

.. skip: end


Resetting offsets
~~~~~~~~~~~~~~~~~
Jobs use offsets to track the last processed data before they were completed or canceled.
In certain cases, it may be necessary to reset a job’s offset to reprocess data.
In the UI, you can reset job offset by selecting **Restart from initial offset** from edit job screen.

.. image:: ../../_static/images/jobs/reset_job_offsets.jpeg
    :alt: Screenshot of resetting Job offsets
    :align: center
    :width: 100%

|

Job offsets can be reset using :py:meth:`~ibm_watsonx_data_integration.cpd_models.job_model.Job.reset_offset`
method on the job instance.

.. warning::
   This method only applies to jobs created from StreamSets flows.

.. code-block:: python

    >>> job.reset_offset()
    <Response [200]>


Deleting a Job
~~~~~~~~~~~~~~

In the UI, you can delete a job by selecting its title from the jobs list.
To delete a job, click the trash icon in the top bar.

.. image:: ../../_static/images/jobs/job_details_page_delete_job.jpeg
   :alt: Screenshot of the Job Details page with the trash icon highlighted
   :align: center
   :width: 100%

|

To delete a job, you can pass the job instance to :py:meth:`Project.delete_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_job>`.
This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> project.delete_job(job)
    <Response [204]>
