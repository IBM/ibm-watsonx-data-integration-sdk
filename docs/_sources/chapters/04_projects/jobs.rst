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
    * Creating and deleting jobs
    * Updating a job
    * Starting a job
    * Cancelling a job run
    * Retrieving logs from a job run

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

You must specify a reference to :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow` object.
Additionally, you can provide optional configuration such as environment variables or job parameters.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.job_model.Job` object.

.. code-block:: python

    >>> job = project.create_job(
    ...     flow=flow,
    ...     name="Test Job",
    ...     description="...",
    ...     job_parameters={"name": "value"}
    ... )
    Job(name='Job for Test flow 1750236869686' version=0, project_name='Test project')

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

    >>> # Returns first job matching given `job_id`
    >>> job = project.jobs.get(job_id="ae0d053b-c4f6-4266-9b02-724e6eb94855")
    Job(name='Job for Test flow 1750236869686' version=0, project_name='Test project')
    >>> # Return a list of all jobs that match `job_type`
    >>> jobs = project.jobs.get_all(job_type="data_intg_flow")
    [Job(name='Job for Test flow 1750236869686' version=0, project_name='Test project')]

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

    >>> job = project.jobs.get(job_id="ae0d053b-c4f6-4266-9b02-724e6eb94855")
    >>> job.name = "New name"
    >>> job.description = "New description."
    >>> res = project.update_job(job)
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

    >>> job = project.jobs.get(job_id="ae0d053b-c4f6-4266-9b02-724e6eb94855")
    >>> res = project.delete_job(job)
    <Response [204]>

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

    >>> job_run = job.start(name="Test Job Run", description="...")
    >>> job_run
    JobRun(name='job run', job_name='Job for Test flow 1750236869686', state='Queued')

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
    >>> job_runs = job.job_runs
    [JobRun(name='job run', job_name='Job for Test flow 1750236869686', state='Queued')]
    >>> # Returns a list of all job runs which status is `Running`
    >>> job_runs = job.job_runs.get_all(states=[JobRunState.Running])
    [JobRun(name='job run', job_name='Job for Test flow 1750236869686', state='Running')]

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

    >>> res = job_run.cancel()
    <Response [204]>

Retrieving a Job Run logs
~~~~~~~~~~~~~~~~~~~~~~~~~

.. TODO Add UI screenshot when logs will be visible in UI

Runtime logs for a job run execution are stored in the
:py:class:`JobRun.logs <ibm_watsonx_data_integration.cpd_models.job_model.JobRun.logs>` property.
It returns a list where each entry is a string containing a log message.

.. code-block:: python

    >>> job_run.logs
    [
        "##I IIS-DSEE-TOSH-00397 2025-05-27 14:27:27(000) Starting job Job for Test flow 1750236869686",
        "##I IIS-DSEE-TOSH-00408 2025-05-27 14:27:27(000) Job Parameters:",
        ...
    ]
