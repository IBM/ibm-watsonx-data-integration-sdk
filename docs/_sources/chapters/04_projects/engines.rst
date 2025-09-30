.. _projects__engines:

Engines
=======

In IBM StreamSets, an engine is a runtime environment where flows are executed.
Settings for engine resources are taken from the configuration of the Environment that the engine belongs to, such as CPU, memory, and storage.
They can be configured with specific settings, like Java version, memory allocation, and maximum concurrent flows, to optimize performance and resource utilization.

Create an Engine
~~~~~~~~~~~~~~~~

The SDK provides functionality for interacting with an engine, however an engine cannot be created directly.
Instead, you can retrieve the installation command to install and start an Engine directly from it's Environment via the :py:meth:`Environment.get_installation_command() <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.get_installation_command>` method.

For more information, refer to our :ref:`Environment <projects__environments>` documentation.

Retrieving Existing Engines in a Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing engines in a project, use the :py:attr:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.engines` property of the
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` class.
This will return a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` objects.

.. code-block:: python

    >>> # Get all engines associated with the project
    >>> engines = project.engines
    >>> engines
    [Engine(..., engine_type='data_collector', ...)]


Retrieving an Existing Engine in a Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing engines in a project, use the :py:meth:`Project.get_engine() <ibm_watsonx_data_integration.cpd_models.project_model.Project.get_engine>` method of the
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` class. You must specify ``engine_id`` of the existing engine.
This will return a :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` object.

In the UI, you can retrieve an Engine ID by navigating to **Manage -> StreamSets -> Environment** and click on your environment name for details.

.. image:: ../../_static/images/engine/retrieve_engine_id.png
   :alt: Screenshot of the Engine id retrieving
   :align: center
   :width: 100%

.. code-block:: python

    >>> # Get a single engine by engine_id
    >>> project.get_engine(engine_id=engines[0].engine_id)
    Engine(..., engine_type='data_collector', ...)


Retrieving Existing Engines in an Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing engines in an Environment, use the :py:attr:`Environment.engines <ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment.engines>` property of the
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.environment_model.Environment` class. This will return a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` objects.

.. code-block:: python

    >>> # Get all engines associated with the environment
    >>> environment.engines
    [Engine(..., engine_type='data_collector', ...)]


Deleting an Existing Engine
~~~~~~~~~~~~~~~~~~~~~~~~~~~
To delete an engine instance, pass the :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.engine_model.Engine` object you want to
delete into the :py:meth:`Project.delete_engine() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_engine>` method to delete it.

.. skip: start "engine is needed for other tests"

.. code-block:: python

    >>> # Delete an engine
    >>> project.delete_engine(engines[0])

.. skip: end
