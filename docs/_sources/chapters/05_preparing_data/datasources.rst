.. _preparing_data__datasources:

.. _projects_preparing_data__datasources_datasource_type:

Datasource Type
===============

A ``datasource type`` is an object that represents a source of data. This can be a database (MongoDB, MySQL), file storage (Dropbox, Google Cloud Storage), generic protocol or API (FTP, HTTP).

.. note::

    For the full list, get all DatasourceTypes objects (See: `Retrieving a DatasourceType`_) or refer to the IBM documentation.

Datasource objects are used as arguments when creating :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection` objects.

.. note::
    To read more about connections see :ref:`Connections <preparing_data__connections>`.

The SDK provides functionality to interact with datasource types.

This includes operations such as:
    * Retrieving DatasourceType(s)

.. _preparing_data__datasources__retrieving_a_datasourcetype:

Retrieving a DatasourceType
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, the list of all DatasourceType objects is displayed during creation of :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection`
(See: :ref:`Creating a Connection <preparing_data__connections__creating_a_connection>`).

.. image:: /_static/images/connections/get_datasource.png
   :alt: Screenshot of the DataSource selection in the UI
   :align: center
   :width: 100%


A DatasourceType can be retrieved using :py:class:`Platform.datasources <ibm_watsonx_data_integration.platform.Platform.datasources>` property.
You can also filter the connections returned based on attributes including
``environment``, ``perspective`` and ``product``.

This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceTypes` object.

.. code-block:: python

    >>> # Return first datasource type matching given `name`
    >>> platform.datasources.get(name='jdbc')
    DatasourceType(name='jdbc')

    >>> # Return first datasource type matching given `perspective`
    >>> platform.datasources.get(perspective='wx')
    DatasourceType(name='informix')

    >>> # Return a list of all datasource types that match `perspective`
    >>> platform.datasources.get_all(perspective='wx')
    [DatasourceType(name='informix'), DatasourceType(name='postgresql-ibmcloud'), ...]

    >>> # Return a list of all datasource types
    >>> platform.datasources
    [DatasourceType(name='informix'), DatasourceType(name='postgresql-ibmcloud'), ...]

.. tip::

    For detailed information about parameters and values refer to https://cloud.ibm.com/apidocs/data-ai-common-core **Datasource Types** section.


.. _preparing_data__datasources__retrieving_a_datasourcetype_connection_properties:

Retrieving a DatasourceType connection properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, available connection fields depend on previously selected DatasourceType (See: :ref:`Retrieving a DatasourceType <preparing_data__datasources__retrieving_a_datasourcetype>`).

For example, JDBC connection by default requires 3 parameters - ``JDBC Connection String``, ``Username``, ``Password``.

.. image:: /_static/images/connections/datasource_jdbc.png
   :alt: Screenshot of the JDBC DataSource arguments.
   :align: center
   :width: 100%

HTTP connection on the other hand by default requires only one parameter - ``File URL``.

.. image:: /_static/images/connections/datasource_http.png
   :alt: Screenshot of the HTTP DataSource arguments.
   :align: center
   :width: 100%

In the SDK to get all properties available for a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceType`
object use :py:class:`DatasourceType.properties <ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceType.properties>`
property (which will return :py:class:`DatasourceTypeProperties <ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceTypeProperties>` object).
Then you can call :py:class:`DatasourceTypeProperties.connection <ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceTypeProperties.connection>`
to retrieve all available connection properties.

.. code-block:: python

    >>> # Get datasource object
    >>> datasource = platform.datasources.get(name='informix')
    >>> datasource
    DatasourceType(name='informix')
    >>> datasource.properties.connection
    [DatasourceTypeProperty(name='cluster_access_token'), DatasourceTypeProperty(name='cluster_user_name'), ...]

    >>> # You can get list of all required properties
    >>> required_properties = datasource.required_connection_properties
    >>> required_properties
    [DatasourceTypeProperty(name='host'), DatasourceTypeProperty(name='password'), ...]

    >>> # Or get details about the property
    >>> required_property_1 = required_properties[0]
    >>> required_property_1
    DatasourceTypeProperty(name='host')
    >>> required_property_1.type
    'string'
    >>> required_property_1.label
    'Hostname or IP address'
