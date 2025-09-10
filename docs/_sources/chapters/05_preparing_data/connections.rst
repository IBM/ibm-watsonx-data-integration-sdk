.. _preparing_data__connections:

Connections
===========
|

A connection is an object that is used to store connection related data for a single datasource (Kafka, Azure, etc.) like credentials, secrets and urls.
Each connection is defined for exactly one datasource.

.. tip::
    To read more about datasources see :ref:`Retrieving an DatasourceType <preparing_data__datasources__retrieving_an_datasourcetype>`.

The SDK provides functionality to interact with connections.

This includes operations such as:
    * Creating a connection
    * Retrieving connection(s)
    * Updating a connection
    * Deleting a connection

.. _preparing_data__connections__creating_a_connection:

Creating a Connection
~~~~~~~~~~~~~~~~~~~~~

In the UI, you can create a new Connection by navigating to **Assets -> New asset -> Connect to a datasource**.

.. image:: ../../_static/images/connections/create_connection.png
   :alt: Screenshot of the Connection creation in the UI - Step 1
   :align: center
   :width: 100%

.. image:: ../../_static/images/connections/create_connection2.png
   :alt: Screenshot of the Connection creation in the UI - Step 2
   :align: center
   :width: 100%

You will need to choose the desired **datasource** from the list

.. image:: ../../_static/images/connections/create_connection3.png
   :alt: Screenshot of the Connection creation in the UI - Step 3
   :align: center
   :width: 100%

and provide a **Name** and other additional configuration (depending on the selected datasource).

.. image:: ../../_static/images/connections/create_connection4.png
   :alt: Screenshot of the Connection creation in the UI - Step 4
   :align: center
   :width: 100%

In the SDK, you can create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection` object within a
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project`,
by selecting the appropriate project from the :py:class:`~ibm_watsonx_data_integration.platform.Platform` and then using
the :py:meth:`Project.create_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_connection>` method to instantiate the connection.


You must provide a ``name`` and ``datasource`` type to create connection.
:py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceType` is an object defining the data source for which the connection will be created.

By default before creating any connection, server will validate it.
If the connection cannot be estabilished with provided parameters, you will get an error and connection will not be saved.
To skip validation and create connection anyway set ``test`` parameter of
:py:meth:`Project.create_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_connection>` to ``False``.

.. note::

    To get available datasources see: :ref:`Retrieving a DatasourceType <preparing_data__datasources__retrieving_an_datasourcetype>`.


.. code-block:: python

    >>> project = platform.projects.get(name="project")
    Project(guid='d7831458-4771-4ad4-bb05-fb95fa94361c', name='project')
    >>> datasource_type = platform.datasources.get(name='http')
    DatasourceType(name='http')
    >>> connection = project.create_connection(
    ...     name="Connection Name",
    ...     datasource_type=datasource_type,
    ...     description="Description ...",
    ...     properties={"url": "https://my/connection/url"},
    ... )
    Connection(name='Connection Name')

.. important::

    Each datasource_type defines its own set of required properties.

    To get list of available properties see: :ref:`Retrieving a DatasourceType connection properties <preparing_data__datasources__retrieving_a_datasourcetype_connection_properties>`.


.. _preparing_data__connections__retrieving_an_existing_connection:

Retrieving an Existing Connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can get all Connections by navigating to **Assets -> Data access -> Connections**.

.. image:: ../../_static/images/connections/get_connections.png
   :alt: Screenshot of the Connection listing in the UI
   :align: center
   :width: 100%

In the SDK, Connection can be retrieved using :py:class:`Project.connections <ibm_watsonx_data_integration.cpd_models.project_model.Project.connections>` property.
You can also further filter and refine the connections returned based on attributes including
``name``, ``context``, ``properties`` and ``datasource_type``.

This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connections` object.

.. code-block:: python

    >>> # Returns the first connection matching given `name`
    >>> connection = project.connections.get(name="Connection Name")
    Connection(name='Connection Name')

    >>> # Return a list of all connections that match `properties`
    >>> connections = project.connections.get_all(
    ...     properties={"url": "https://my/connection/url"}
    ... )
    [Connection(name='Connection Name')]

    >>> # Return a list of all connections
    >>> connections = project.connections
    [Connection(name='Connection Name'), (...)]

.. tip::

    For detailed information about parameters and values refer to https://cloud.ibm.com/apidocs/data-ai-common-core#listconnections.

.. _preparing_data__connections__updating_a_connection:

Updating a Connection
~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can update a Connection by navigating to **Assets -> Data access -> Connections** and clicking the edit button on the Connection you want to edit.
This button is visible only when cursor is over the Connection object.

.. image:: ../../_static/images/connections/update_connection.png
   :alt: Screenshot of the Connection updating in the UI
   :align: center
   :width: 100%

To update a connection in the SDK, we first modify properties of the connection and then
pass the instance to the :py:meth:`Project.update_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_connection>` method.

This method returns an HTTP response indicating the status of the update operation.

Similar to :ref:`Creating a Connection <preparing_data__connections__creating_a_connection>` you can skip the validation by setting ``test`` parameter of
:py:meth:`Project.update_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_connection>` to ``False``.

.. code-block:: python

    >>> connection = project.connections.get(name="Connection Name")
    Connection(name='Connection Name')
    >>> connection.name = "New Connection Name"
    >>> res = project.update_connection(connection)
    <Response [200]>

.. _preparing_data__connections__deleting_a_connection:

Deleting a Connection
~~~~~~~~~~~~~~~~~~~~~

In the UI, you can delete an existing Connection by navigating to the **Assets -> Data access -> Connections**.

.. image:: ../../_static/images/connections/delete_connection.png
   :alt: Screenshot of the Connection deletion in the UI
   :align: center
   :width: 100%

In the SDK to delete a connection instance you can pass it to :py:meth:`Project.delete_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_connection>` method.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> connection = project.connections.get(name="New Connection Name")
    Connection(name='New Connection Name')
    >>> res = project.delete_connection(connection)
    <Response [204]>
