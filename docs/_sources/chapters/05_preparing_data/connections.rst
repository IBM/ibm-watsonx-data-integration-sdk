.. _preparing_data__connections:

Connections
===========

A connection is an object used to store connection-related data for a single datasource (Kafka, Azure, etc.), such as credentials, secrets, and URLs.
Each connection is defined by exactly one datasource.

.. tip::
    To read more about datasources see :ref:`Retrieving a DatasourceType <preparing_data__datasources__retrieving_a_datasourcetype>`.

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

.. image:: /_static/images/connections/create_connection.png
   :alt: Screenshot of the Connection creation in the UI - Step 1
   :align: center
   :width: 100%

.. image:: /_static/images/connections/create_connection2.png
   :alt: Screenshot of the Connection creation in the UI - Step 2
   :align: center
   :width: 100%

You will need to choose the desired **datasource** from the list.

.. image:: /_static/images/connections/create_connection3.png
   :alt: Screenshot of the Connection creation in the UI - Step 3
   :align: center
   :width: 100%

and provide a **Name** and other additional configuration (depending on the selected datasource).

.. image:: /_static/images/connections/create_connection4.png
   :alt: Screenshot of the Connection creation in the UI - Step 4
   :align: center
   :width: 100%

In the SDK, you can create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection` object within a
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project`,
by selecting the appropriate project from the :py:class:`~ibm_watsonx_data_integration.platform.Platform` and then using
the :py:meth:`Project.create_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_connection>` method to instantiate the connection.


You must provide a ``name`` and ``datasource_type`` to create a connection.
:py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceType` is an object that defines the data source for which the connection will be created.

By default, the server validates a connection before creating it.
If the connection cannot be established with the provided parameters, you will get an error and the connection will not be saved.
To skip validation and create the connection anyway, set the ``test`` parameter of
:py:meth:`Project.create_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_connection>` to ``False``.

.. note::

    To see available datasources, refer to :ref:`Retrieving a DatasourceType <preparing_data__datasources__retrieving_a_datasourcetype>`.


.. code-block:: python

    >>> datasource_type = platform.datasources.get(name='http')
    >>> connection = project.create_connection(
    ...     name='Connection Name',
    ...     datasource_type=datasource_type,
    ...     description='Description ...',
    ...     properties={'url': 'https://my/connection/url'},
    ... )
    >>> connection
    Connection(name='Connection Name')

.. important::

    Each ``datasource_type`` defines its own set of required properties.

    To see the list of available properties, refer to :ref:`Retrieving a DatasourceType connection properties <preparing_data__datasources__retrieving_a_datasourcetype_connection_properties>`.


.. _preparing_data__connections__retrieving_an_existing_connection:

Retrieving an existing Connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can view all Connections by navigating to **Assets -> Data access -> Connections**.

.. image:: /_static/images/connections/get_connections.png
   :alt: Screenshot of the Connection listing in the UI
   :align: center
   :width: 100%

In the SDK, a Connection can be retrieved using the :py:class:`Project.connections <ibm_watsonx_data_integration.cpd_models.project_model.Project.connections>` property.
You can also filter the returned connections based on attributes including
``name``, ``connection_id``, ``context``, ``properties``, and ``datasource_type``.

This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connections` object.

.. code-block:: python

    >>> # Returns the first connection matching given `name`
    >>> connection = project.connections.get(name='Connection Name')
    >>> connection
    Connection(name='Connection Name')

    >>> # Get the connection id
    >>> connection_id = connection.connection_id

    >>> # Returns a connection by its connection_id
    >>> project.connections.get(connection_id=connection_id)
    Connection(name='Connection Name')

    >>> # Return a list of all connections that match `properties`
    >>> project.connections.get_all(
    ...     properties={'url': 'https://my/connection/url'}
    ... )
    [Connection(name='Connection Name')]

    >>> # Return a list of all connections
    >>> project.connections # doctest: +SKIP
    [Connection(name='Connection Name')]

.. tip::

    For detailed information about parameters and values refer to https://cloud.ibm.com/apidocs/data-ai-common-core **Connections** section.

.. _preparing_data__connections__updating_a_connection:

Updating a Connection
~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can update a Connection by navigating to **Assets -> Data access -> Connections** and clicking the edit button on the Connection you want to edit.
This button is visible only when the cursor is over the Connection object.

.. image:: /_static/images/connections/update_connection.png
   :alt: Screenshot of the Connection updating in the UI
   :align: center
   :width: 100%

To update a connection in the SDK, first modify the properties of the connection and then
pass the instance to the :py:meth:`Project.update_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_connection>` method.

This method returns an HTTP response indicating the status of the update operation.

Similar to :ref:`Creating a Connection <preparing_data__connections__creating_a_connection>`, you can skip validation by setting the ``test`` parameter of
:py:meth:`Project.update_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_connection>` to ``False``.

.. code-block:: python

    >>> connection.name = 'New Connection Name'
    >>> project.update_connection(connection)
    <Response [200]>
    >>> project.connections.get(name='New Connection Name') # doctest: +SKIP
    Connection(name='New Connection Name')

.. _preparing_data__connections__using_a_connection:

Using a Connection
~~~~~~~~~~~~~~~~~~

In the UI, you can use a Connection in a flow by adding a stage that takes a Connection and choosing the Connection in the view connection or connection property.
For batch flows, this would be any connector stage.

.. image:: /_static/images/connections/use_connection_batch.png
   :alt: Screenshot of the batch Connection usage in the UI
   :align: center
   :width: 100%

For streaming flows, this would be select source and target stages.

.. image:: /_static/images/connections/use_connection_streaming.png
   :alt: Screenshot of the streaming Connection usage in the UI
   :align: center
   :width: 100%

In the SDK, to use a Connection instance for batch flows, you can pass it to the :py:meth:`StageNode.use_connection() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.StageNode.use_connection>` method of the connector stage.
To use a Connection instance for streaming flows, you can pass it to the :py:meth:`Stage.use_connection() <ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stage.use_connection>` method of the target or destination stage.

.. code-block:: python

    >>> # Batch Flow
    >>> new_flow = project.create_flow(name='New flow', environment=None, flow_type='batch')
    >>> connection = project.connections.get(name='New Connection Name')
    >>> connection # doctest: +SKIP
    Connection(name='New Connection Name')
    >>> http = new_flow.add_stage(type='HTTP', label='HTTP_Stage_1')
    >>> http.use_connection(connection) # doctest: +SKIP
    >>> # Streaming Flow
    >>> connection_to_use
    Connection(name='dummy_connection ...')
    >>> origin = flow.add_stage('JDBC Query Consumer')
    >>> origin.use_connection(connection_to_use)

.. invisible-code-block: python

    >>> flow.remove_stage(origin)


.. _preparing_data__connections__using_a_local_connection:

Using a local Connection (Batch Only)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An alternative way to use a connector stage is to put connection parameters directly into the stage's properties. In the UI, this can be done under the **Connection details** section of the stage.

If you are already :ref:`Using a Connection <preparing_data__connections__using_a_connection>`, the **Connection details** section will not be visible.

.. image:: /_static/images/connections/use_local_connection.png
   :alt: Screenshot of the Local Connection usage in the UI
   :align: center
   :width: 100%

In the SDK, each connector stage's configuration has a connection property. To use a local connection, you can directly edit the desired fields of this connection property.

.. code-block:: python

    >>> http = new_flow.add_stage(type='HTTP', label='HTTP_Stage_1')
    >>> http.configuration.connection.url = 'yoururl.com'
    >>> http.configuration.connection.ssl_certificate = 'yoursslcertificate'


.. _preparing_data__connections__deleting_a_connection:

Deleting a Connection
~~~~~~~~~~~~~~~~~~~~~

In the UI, you can delete an existing Connection by navigating to the **Assets -> Data access -> Connections**.

.. image:: /_static/images/connections/delete_connection.png
   :alt: Screenshot of the Connection deletion in the UI
   :align: center
   :width: 100%

In the SDK, to delete a connection instance, you can pass it to the :py:meth:`Project.delete_connection() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_connection>` method.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> project.delete_connection(connection)
    <Response [204]>
