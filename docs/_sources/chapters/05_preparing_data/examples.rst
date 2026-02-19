.. _preparing_data__examples:

Examples
========

.. _preparing_data__examples__batch_flow:

Creating and Running a Batch Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is an example of how to create and run a simple batch flow. The flow will contain a Row Generator stage connected to a Peek stage. Here is a picture of what the flow looks like in the UI.

.. image:: /_static/images/flows/rowgen_peek.png
   :alt: Screenshot of the Rowgen Peek flow.
   :align: center
   :width: 100%


This example will cover 4 steps:
   * Set up authentication
   * Create a new project
   * Create and save a new flow
   * Create and run a job for the flow

.. code-block:: python

   from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
   from ibm_watsonx_data_integration import Platform
   from ibm_watsonx_data_integration.services.datastage import *

   # 1.Set up authentication
   api_key = 'API_KEY'
   auth = IAMAuthenticator(api_key=api_key)
   platform = Platform(auth, base_api_url='https://api.ca-tor.dai.cloud.ibm.com')

   # 2.Create a new project
   project = platform.create_project(
      name='My first project',
      description='Building sample batch flows',
      tags=['flow_test_project'],
      public=True,
      project_type='wx'
   )
   project

   # 3.Create and save a new flow
   # Flow
   flow = project.create_flow(
      name='RowGenPeek',
      environment=None,
      flow_type='batch'
   )
   # Stages
   row_generator = flow.add_stage('Row Generator', 'Row_Generator')
   peek = flow.add_stage('Peek', 'Peek')
   # Links
   link_1 = row_generator.connect_output_to(peek)
   link_1.name = 'Link_1'
   row_generator_schema = link_1.create_schema()
   row_generator_schema.add_field('VARCHAR', 'COLUMN_1').length(100)

   project.update_flow(flow)

   # 4.Run a job for the flow
   row_gen_peek_job = project.create_job(name='RowGenPeek_job', flow=flow)
   job_start = row_gen_peek_job.start(name='RowGenPeek_job_run', description='')


.. _preparing_data__examples__streaming_flow:

Creating and Running a Streaming Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is an example of how to create and run a simple streaming flow. The flow will contain a Dev Raw Data Source connected to a Trash stage. Here is a picture of what the flow looks like in the UI.

.. image:: /_static/images/flows/devtrash.png
   :alt: Screenshot of the Dev Raw Data Source Trash flow.
   :align: center
   :width: 100%


This example will cover 6 steps:
   * Set up authentication
   * Create a new project
   * Create a new environment
   * Create and save a new flow
   * Create and run a job for the flow
   * Retrieve and run the engine installation command

.. code-block:: python

   from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
   from ibm_watsonx_data_integration import Platform
   from ibm_watsonx_data_integration.services.datastage import *

   # 1.Set up authentication
   api_key = 'API_KEY'
   auth = IAMAuthenticator(api_key=api_key)
   platform = Platform(auth, base_api_url='https://api.ca-tor.dai.cloud.ibm.com')

   # 2.Create a new project
   project = platform.create_project(
      name='My first project',
      description='Building sample streaming flows',
      tags=['flow_test_project'],
      public=True,
      project_type='wx'
   )

   # 3.Create a new environment
   environment = project.create_environment(
        name='My first environment',
        engine_version=platform.available_engine_versions[0],
    )

   # 4.Create and save a new flow
   # Flow
   flow = project.create_flow(
      name='DevRawDataSourceTrash',
      environment=environment,
      flow_type='streaming'
   )
   # Stages
   dev = flow.add_stage('Dev Raw Data Source')
   trash = flow.add_stage('Trash')

   # Links
   dev.connect_output_to(trash)

   project.update_flow(flow)

   # 5.Run a job for the flow
   dev_trash_job = project.create_job(name='DevRawDataSourceTrash_job', flow=flow)
   job_start = dev_trash_job.start(name='DevRawDataSourceTrash_job_run', description='')

Before this flow can be run there is one more step. You must retrieve the installation command for the engine of the environment by calling the :py:meth:`Environment.get_installation_command() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.get_installation_command>` method.
Then, replace the ``SSET_API_KEY`` variable in the command with your ``api_key`` and run the command.

.. code-block:: python

   # Replace the returned command's SSET_API_KEY with your api_key and then run the command.
   environment.get_installation_command()
