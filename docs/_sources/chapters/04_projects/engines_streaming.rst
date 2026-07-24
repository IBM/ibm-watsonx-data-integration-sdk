.. _projects__engines:

Engines (Streaming)
===================

Engines are supported only for streaming flows.
In IBM StreamSets, an engine is a runtime environment where flows are executed.
Settings for engine resources are taken from the configuration of the environment to which the engine belongs, such as CPU, memory, and storage.
Engines can be configured with specific settings, such as Java version, memory allocation, and maximum concurrent flows, to optimize performance and resource utilization.

Creating an Engine
~~~~~~~~~~~~~~~~~~

The SDK provides functionality for interacting with an engine; however, an engine cannot be created directly.
Instead, you can retrieve the installation command to install and start an engine directly from its environment via the :py:meth:`Environment.get_installation_command() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.get_installation_command>` method.

For more information, refer to our :ref:`Environment <projects__environments_streaming>` documentation.

Retrieving Existing Engines in a Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing engines in a project, use the :py:attr:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.engines` property of the
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` class.
This will return a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` objects.
You can also retrieve a single engine using the :py:meth:`Project.engines.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method, which takes unique identifiers such as engine_type or engine_id.

.. code-block:: python

    >>> # Get all engines associated with the project
    >>> engines = project.engines
    >>> engines
    [Engine(..., engine_type='data_collector', ...)]
    >>> engine = project.engines.get(engine_type='data_collector')
    >>> engine
    Engine(..., engine_type='data_collector', ...)


Retrieving Existing Engines in an Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing engines in an Environment, use the :py:attr:`Environment.engines <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.engines>` property of the
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment` class. This will return a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` objects.

.. code-block:: python

    >>> # Get all engines associated with the environment
    >>> environment.engines
    [Engine(..., engine_type='data_collector', ...)]


.. _projects__engines__engine_commands:

Engine Commands
~~~~~~~~~~~~~~~
The SDK supports restart and shutdown commands for an engine at three places:

* On the :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine`
  through :py:meth:`Engine.restart() <ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine.restart>` and
  :py:meth:`Engine.shutdown() <ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine.shutdown>`.
* In an :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment`
  through :py:meth:`Environment.restart_all_engines() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.restart_all_engines>` and
  :py:meth:`Environment.shutdown_all_engines() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.shutdown_all_engines>`.
* In a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project`
  through :py:meth:`Project.restart_streaming_engines() <ibm_watsonx_data_integration.cpd_models.project_model.Project.restart_streaming_engines>` and
  :py:meth:`Project.shutdown_streaming_engines() <ibm_watsonx_data_integration.cpd_models.project_model.Project.shutdown_streaming_engines>`.

Restarting and Shutting Down a Single Engine
--------------------------------------------
Use the engine instance directly when you want to target one engine.

.. skip: start 'engine is required for other tests'

.. code-block:: python

    >>> engine = environment.engines[0]
    >>> engine.restart()
    <Response [200]>
    >>> engine.shutdown()
    <Response [200]>

.. skip: end

Restarting and Shutting Down All Engines in an Environment
----------------------------------------------------------
Use the environment-level commands when you want to operate on every engine associated with one environment.

.. skip: start 'engine is required for other tests'

.. code-block:: python

    >>> environment.restart_all_engines()
    <Response [200]>
    >>> environment.shutdown_all_engines()
    <Response [200]>

.. skip: end

Restarting and Shutting Down Engines from a Project
---------------------------------------------------
Use the project-level commands when you want to restart or shut down one or more specific engines in the project.

.. skip: start 'engine is required for other tests'

.. code-block:: python

    >>> engine = environment.engines[0]
    >>> project.restart_streaming_engines(engine)
    <Response [200]>
    >>> project.shutdown_streaming_engines(engine)
    <Response [200]>

.. skip: end

The project-level command methods require at least one engine argument. Passing no engines raises :py:class:`ValueError`.

Deleting an Existing Engine
~~~~~~~~~~~~~~~~~~~~~~~~~~~
To delete a single engine instance, pass the :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` object you want to
delete into the :py:meth:`Project.delete_engine() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_engine>` method to delete it.

.. skip: start 'engine is needed for other tests'

.. code-block:: python

    >>> # Delete a single engine
    >>> project.delete_engine(engines[0])

.. skip: end

Deleting Multiple Engines
~~~~~~~~~~~~~~~~~~~~~~~~~
To delete multiple engines at once, use the :py:meth:`Project.delete_engines() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_engines>`
method. This performs a bulk delete operation, which is more efficient than deleting engines individually.

.. skip: start 'engines are needed for other tests'

.. code-block:: python

    >>> # Delete multiple engines at once
    >>> project.delete_engines(engines[0], engines[1], engines[2])
    <Response [200]>

    >>> # Delete all engines from an environment
    >>> environment = project.environments.get(environment_id='environment-id')
    >>> project.delete_engines(*environment.engines.get_all())
    <Response [200]>

.. skip: end
