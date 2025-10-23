.. _projects__environments:

Environments
============

An environment defines the execution context for running flows.
It specifies the engine to be used, the libraries to be installed, and additional runtime parameters such as the number of CPU cores, memory allocation, and the maximum number of concurrent flow runs.

When configuring an environment, a command is generated that can be used to launch the environment.
This command typically runs a Docker container with the selected engine version, all required libraries, and the specified parameters.

.. _projects__environments__creating_an_environment:

Creating an Environment
~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can create a new Environment by navigating to **Manage -> StreamSets -> New environment.**

.. image:: ../../_static/images/environment/create_environment.png
   :alt: Screenshot of the Environment creation form in the UI
   :align: center
   :width: 100%

You will be prompted to provide a **Name** and select a **Data Collector engine version**, and other additional configuration options.

.. image:: ../../_static/images/environment/create_environment2.png
   :alt: Screenshot of the Environment configuration form in the UI
   :align: center
   :width: 100%

In the SDK, you can create an environment from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_environment() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_environment>` method.
At a minimum, you must provide the ``name`` and ``engine_version`` parameters.
All other parameters are optional and will be populated with default values if not specified.
This method returns a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment` object.

.. code-block:: python

    >>> env = project.create_environment(
    ...     name='Sample',
    ...     # Optional parameters - see API Reference for more optional parameters
    ...     description='Basic env.',
    ...     stage_libs=[
    ...         'streamsets-datacollector-basic-lib',
    ...         'streamsets-datacollector-dataformats-lib',
    ...         'streamsets-datacollector-dev-lib'
    ...     ],
    ...     cpus_to_allocate=2,
    ... )
    >>> env
    Environment(name='Sample', description='Basic env.', ...)


.. note::

    | For information on available Engine versions, see :ref:`Engines Versions <retrieving_available_engine_versions>`.
    | For details on supported libraries, see :ref:`Engine Details <retrieving_engine_version_details>`.


Retrieving an Existing Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Environments can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object.
You can access all environments associated with a project using the
:py:attr:`Project.environments <ibm_watsonx_data_integration.cpd_models.project_model.Project.environments>` property.
You can also retrieve single environment using the :py:meth:`Project.environments.get()` method,
which requires the ``environment_id`` parameter.

.. code-block:: python

    >>> # Get all environments associated with the project
    >>> project.environments
    [...Environment(name='Sample', description='Basic env.', ...)...]

    >>> # Get a single environment by its id
    >>> project.environments.get(environment_id=env.environment_id)
    Environment(name='Sample', description='Basic env.', ...)


Modifying an Environment
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can update or delete an existing Environment by navigating to the **Manage -> StreamSets**.

.. image:: ../../_static/images/environment/edit_environment.png
   :alt: Screenshot of the Environment update form in the UI
   :align: center
   :width: 100%


Updating an Environment
-----------------------

Similar to environment creation, environment can also be updated using the :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object.
First, modify properties of the environment instance, then update it using the
:py:meth:`Project.update_environment() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_environment>` method.

.. code-block:: python

    >>> # Modify environment settings
    >>> env.max_memory_used = 80
    >>> env.stage_libs.append('streamsets-datacollector-aws-lib')

    >>> # Update the environment on the platform
    >>> project.update_environment(env)
    <Response [200]>
    >>> env = project.environments.get(environment_id=env.environment_id)
    >>> env.stage_libs
    ['streamsets-datacollector-basic-lib', 'streamsets-datacollector-dataformats-lib', 'streamsets-datacollector-dev-lib', 'streamsets-datacollector-aws-lib']

.. _projects__environments__retrieving_install_command:

Retrieving the Engine Installation Command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To start and run the Engine defined by an Environment, you'll need to retrieve the installation command for that Environment and execute it from the machine where you want the Engine to run.

In the UI, you can retrieve the run command by navigating to the **Manage -> StreamSets**.

.. image:: ../../_static/images/environment/get_run_command.png
   :alt: Screenshot of the Environment update form in the UI
   :align: center
   :width: 100%

You can retrieve the run command via the :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment` object by using
:py:meth:`Environment.get_installation_command() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.get_installation_command>` method.

.. skip: start "docker command output in non parsable format"

.. code-block:: python

    >>> env.get_installation_command(
    ...     # Optional parameters
    ...     pretty=False,
    ...     foreground=True
    ... )
    'docker run -d --restart on-failure --cpus=4.0 --hostname "$(hostname)" -p 18630:18630 ...'

.. skip: end

.. note::
   Please be aware that the installation command you retrieve from the Environment requires the ``SSET_API_KEY`` environment variable to be set for the user executing the command.
   The environment variable should contain the API key you generated for :ref:`authenticating <getting_started_and_tutorials__authentication>` with the watsonx.data integration platform.


Deleting an Environment
-----------------------

To remove an environment, use the
:py:meth:`Project.delete_environment() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_environment>` method.
The delete method returns an API response, which you can insepct to verify the status code.

.. code-block:: python

    >>> project.delete_environment(env)
    <Response [200]>


.. _retrieving_available_engine_versions:

Retrieving Available Engine Versions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To view the list of available Engine versions, use the
:py:attr:`Platform.available_engine_versions <ibm_watsonx_data_integration.platform.Platform.available_engine_versions>` property.
This returns a list of Engine version names that can be used when configuring an Environment.

.. code-block:: python

    >>> platform.available_engine_versions
    [...'JDK17_6.3.0'...]


.. _retrieving_engine_version_details:

Retrieving Engine Version Details
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can view the list of libraries supported by a specific Engine version during Environment configuration:

.. image:: ../../_static/images/environment/engine_libraries.png
    :alt: Screenshot showing supported libraries for an Engine version in the UI
    :align: center
    :width: 100%

In the SDK, to retrieve the list of libraries supported by a given Engine version, use the
:py:meth:`Platform.get_engine_version_info() <ibm_watsonx_data_integration.platform.Platform.get_engine_version_info>` method.
This method takes the engine version name as an argument and returns detailed information about that version, including its supported libraries.

.. code-block:: python

    >>> engine_info = platform.get_engine_version_info('JDK17_6.3.0')
    >>> engine_info
    {'engine_version_id': 'JDK17_6.3.0', ...'engine_type': 'data_collector', 'stage_libs': [...]...}

    >>> # To see all stage library IDs that can be used in environment.stage_libs
    >>> [lib['stage_lib_id'] for lib in engine_info.get('stage_libs')]
    [...'streamsets-datacollector-aws-lib', 'streamsets-datacollector-apache-kafka-lib'...]
