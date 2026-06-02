.. _preparing_data__pipelines:

Pipelines
=========

A pipeline is an orchestration object used to coordinate and execute complex data workflows. Pipelines are built using Kubeflow Pipelines (KFP) and allow you to define workflows as Python functions decorated with ``@dsl.pipeline``. Unlike batch or streaming flows, pipelines focus on orchestrating multiple tasks, jobs, and components in a coordinated manner.

The SDK provides the following functionality to interact with pipelines:
    * Creating a pipeline
    * Retrieving pipelines
    * Duplicating a pipeline
    * Deleting a pipeline
    * Working with pipeline components

.. _preparing_data__pipelines__creating_a_pipeline:

Creating a Pipeline
~~~~~~~~~~~~~~~~~~~

In the UI, you can create a pipeline by navigating to **Assets -> New asset -> Automate model lifecycles with Pipelines**.

.. image:: /_static/images/pipelines/create_pipeline.png
    :alt: Screenshot of the pipeline creation page.
    :align: center
    :width: 100%

In the SDK, you can create a pipeline from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_pipeline() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_pipeline>` method.
You are required to supply a ``name`` parameter, a ``pipeline_function`` (a KFP pipeline decorated with ``@dsl.pipeline``), and optional ``description`` and ``parameter_sets`` parameters.
This method returns a :py:class:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_model.Pipeline` instance.

.. skip: start

.. code-block:: python

    >>> from kfp import dsl
    >>>
    >>> @dsl.component
    ... def add_two_numbers(a: int, b: int) -> int:
    ...     print(f"Adding numbers: {a} + {b}")
    ...     return a + b
    >>>
    >>> @dsl.pipeline
    ... def my_pipeline() -> None:
    ...     add_two_numbers(a=1, b=2)
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='My first pipeline',
    ...     pipeline_function=my_pipeline,
    ...     description='A simple addition pipeline'
    ... )
    >>> pipeline
    Pipeline(pipeline_id='...', name='My first pipeline')

.. skip: end

.. note::
    Pipelines use Kubeflow Pipelines (KFP) syntax. You must define your pipeline using the ``@dsl.pipeline`` decorator and components using the ``@dsl.component`` decorator.

.. _preparing_data__pipelines__retrieving_a_pipeline:

Retrieving Pipelines
~~~~~~~~~~~~~~~~~~~~

Pipelines can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.pipelines <ibm_watsonx_data_integration.cpd_models.project_model.Project.pipelines>` property.
You can retrieve a single pipeline using the :py:meth:`Pipelines.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method which takes in unique identifiers such as the ``pipeline_id`` or ``name``.

.. code-block:: python

    >>> # Get all pipelines
    >>> project.pipelines
    [Pipeline(pipeline_id='...', name='My first pipeline', description='A test pipeline for documentation')]

    >>> # Get a specific pipeline by name
    >>> my_pipeline = project.pipelines.get(name='My first pipeline')
    >>> my_pipeline
    Pipeline(pipeline_id='...', name='My first pipeline', ...)

    >>> # Get a specific pipeline by ID
    >>> project.pipelines.get(pipeline_id=my_pipeline.pipeline_id)
    Pipeline(pipeline_id='...', name='My first pipeline', ...)

.. _preparing_data__pipelines__duplicating_a_pipeline:

Duplicating a Pipeline
~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can duplicate a pipeline by navigating to **Assets**, finding your pipeline, clicking the three dots, and selecting **Duplicate**.

.. image:: /_static/images/pipelines/duplicate_pipeline.png
    :alt: Screenshot of duplicating a pipeline.
    :align: center
    :width: 100%

To duplicate a pipeline using the SDK, pass a :py:class:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_model.Pipeline` instance
to the :py:meth:`Project.duplicate_pipeline() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_pipeline>` method,
along with the ``name`` parameter for the new pipeline and an optional ``description`` parameter.

This duplicates the pipeline and returns a new instance of :py:class:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_model.Pipeline`.

.. code-block:: python

    >>> duplicated_pipeline = project.duplicate_pipeline(
    ...     pipeline,
    ...     name='My duplicated pipeline',
    ...     description='A copy of my first pipeline'
    ... )
    >>> duplicated_pipeline
    Pipeline(pipeline_id='...', name='My duplicated pipeline', ...)

.. _preparing_data__pipelines__deleting_a_pipeline:

Deleting a Pipeline
~~~~~~~~~~~~~~~~~~~

In the UI, you can delete a pipeline by navigating to **Assets**, finding your pipeline, clicking the three dots, and selecting **Delete**.

.. image:: /_static/images/pipelines/delete_pipeline.png
    :alt: Screenshot of deleting a pipeline.
    :align: center
    :width: 100%

To delete a pipeline using the SDK, pass a :py:class:`~ibm_watsonx_data_integration.services.pipelines.models.pipeline_model.Pipeline` instance to the :py:meth:`Project.delete_pipeline() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_pipeline>` method.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> response = project.delete_pipeline(duplicated_pipeline)
    >>> response.status_code
    204

.. _preparing_data__pipelines__running_a_pipeline:

Running a Pipeline
~~~~~~~~~~~~~~~~~~

To run a pipeline, you first need to create a job from the pipeline, then start the job.
This is done using the :py:meth:`Project.create_job() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_job>` method
followed by the :py:meth:`Job.start() <ibm_watsonx_data_integration.cpd_models.job_model.job_model.Job.start>` method.

.. skip: start

.. code-block:: python

    >>> # Create a job for the pipeline
    >>> pipeline_job = project.create_job(name='My pipeline job', flow=pipeline)
    >>> pipeline_job
    Job(name='My pipeline job', ...)

    >>> # Start the job
    >>> job_run = pipeline_job.start(name='My pipeline job run', description='First run')
    >>> job_run
    JobRun(...)

    >>> # Check the status
    >>> job_run.state
    'Running'

.. skip: end

.. _preparing_data__pipelines__using_parameter_sets:

Using Parameter Sets with Pipelines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pipelines can use parameter sets to make them more flexible and reusable. Parameter sets allow you to define parameters that can be referenced within your pipeline.

.. skip: start

.. code-block:: python

    >>> from ibm_watsonx_data_integration.cpd_models.parameter_set_model import ParameterSet, ParameterType
    >>>
    >>> # Create a parameter set
    >>> param_set = project.create_parameter_set(
    ...     name='pipeline_params',
    ...     parameters=[
    ...         {'name': 'input_value', 'type': ParameterType.String, 'value': 'test'},
    ...         {'name': 'threshold', 'type': ParameterType.Integer, 'value': 100}
    ...     ]
    ... )
    >>>
    >>> # Create a pipeline with parameter sets
    >>> @dsl.pipeline
    ... def my_pipeline_with_params() -> None:
    ...     add_two_numbers(a=10, b=20)
    >>>
    >>> pipeline = project.create_pipeline(
    ...     name='Pipeline with params',
    ...     pipeline_function=my_pipeline_with_params,
    ...     parameter_sets=[param_set]
    ... )

.. skip: end
