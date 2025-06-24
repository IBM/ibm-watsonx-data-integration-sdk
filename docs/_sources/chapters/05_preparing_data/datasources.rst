.. _preparing_data__datasources:

.. _projects_preparing_data__datasources_datasource_type:

Datasource Type
===============
|

A ``datasource type`` is an object that represents a source of the data. This can be a database (MongoDB, MySQL), file storage (Dropbox, Google Cloud Storage), generic protocol or API (FTP, HTTP).

.. note::

    For the full list, get all DatasourceTypes objects (`Retrieving a DatasourceType`_) or refer to the IBM documentation.

Datasource objects are used as arguments when creating :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection` objects.

.. note::
    To read more about connections see :ref:`Connections <preparing_data__connections>`.

The SDK provides functionality to interact with datasource types.

This includes operations such as:
    * Retrieving DatasourceType(s)

.. _preparing_data__datasources__retrieving_an_datasourcetype:

Retrieving a DatasourceType
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, the list of all DatasourceType objects is displayed during creation of :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection`
(See: :ref:`Creating a Connection <preparing_data__connections__creating_a_connection>`).

In the WatsonX DI SDK, a Connection can be retrieved using :py:class:`Project.connections <ibm_watsonx_data_integration.cpd_models.project_model.Project.connections>` property.
You can also further filter and refine the connections returned based on attributes including
``name``, ``context``, ``properties`` and ``datasource_type``.

A DatasourceType can be retrieved using :py:class:`Platform.datasources <ibm_watsonx_data_integration.platform.Platform.datasources>` property.
You can also further filter and refine the connections returned based on attributes including
``environment``, ``perspective`` and ``product``.

This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceTypes` object.

.. code-block:: python

    >>> # Return first datasource type matching given `perspective`
    >>> datasource = platform.datasources.get(perspective="wx")
    DatasourceType(name='informix')

    >>> # Return a list of all datasource types that match `perspective`
    >>> datasources = platform.datasources.get_all(perspective="wx")
    [
        DatasourceType(name='informix'),
        DatasourceType(name='postgresql-ibmcloud'),
        (...)
    ]

    >>> # Return a list of all datasource types
    >>> datasources = platform.datasources
    [
        DatasourceType(name='informix'),
        DatasourceType(name='postgresql-ibmcloud'),
        (...)
    ]

.. tip::

    For detailed information about parameters and values refer to https://cloud.ibm.com/apidocs/data-ai-common-core#listdatasourcetypes.


.. _preparing_data__datasources__retrieving_a_datasourcetype_connection_properties:

Retrieving a DatasourceType connection properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all properties available for a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.DatasourceType` object use :py:class:`DatasourceType.properties.connection <ibm_watsonx_data_integration.cpd_models.project_model.DatasourceType.properties.connection>` property.

.. code-block:: python

    >>> # Get datasource object
    >>> datasource = platform.datasources[0]
    DatasourceType(name='informix')
    >>> properties = datasource.properties.connection
    [
        DatasourceTypeProperty(name='cluster_access_token'),
        DatasourceTypeProperty(name='cluster_user_name'),
        (...)
    ]

    >>> # You can get list of all required properties
    >>> required_properties = [p for p in properties if p.required]
    [
        DatasourceTypeProperty(name='host'),
        DatasourceTypeProperty(name='password'),
        (...)
    ]

    >>> # Or get details about the property
    >>> reqired_property_1 = required_properties[0]
    DatasourceTypeProperty(name='host')
    >>> required_property_1.type
    'string'
    >>> required_property_1.label
    'Hostname or IP address'
