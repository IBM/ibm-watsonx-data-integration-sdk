Service Instances
=================

A service instance is a provisioned instance of a Cloud Pak for Data service, which allows you to execute and manage workloads (such as jobs and flows) through its dedicated compute resources.

The SDK provides functionality to interact with service instances on the watsonx.data integration platform.

This includes operations such as:
    * Creating Service Instances
    * Retrieving Service Instances
    * Deleting Service Instances

.. note::
   Working with service instances is only supported on Cloud Pak for Data SaaS.


Creating Service Instances
~~~~~~~~~~~~~~~~~~~~~~~~~~
To create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.service_model.Service` object within the watsonx.data integration platform, you can use the :py:meth:`Platform.create_service_instance() <ibm_watsonx_data_integration.platform.Platform.create_service_instance>` method to instantiate the service instance.

You must specify ``instance_type`` and ``name``. Optionally, you can also specify ``target`` and ``tags``.

.. skip: start 'paid plan only'

.. code-block:: python

    >>> service_instance = platform.create_service_instance(
    ...     instance_type='datastage',
    ...     name='My Datastage Service Instance',
    ...     target='us-south',
    ...     tags=['DatastageInstance']
    ... )


Retrieving Service Instances
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing service instances, use the :py:attr:`~ibm_watsonx_data_integration.platform.Platform.service_instances` property of the :py:class:`~ibm_watsonx_data_integration.platform.Platform` class. This will return a :py:class:`~ibm_watsonx_data_integration.cpd_models.service_model.Services` object.

.. code-block:: python

    >>> # Returns a CollectionModel of all service instances
    >>> service_instances = platform.service_instances

    >>> # Returns a CollectionModel of the first five service instances
    >>> service_instances = platform.service_instances.get_all(limit=5)


Deleting Service Instances
~~~~~~~~~~~~~~~~~~~~~~~~~~
To delete a service instance, pass the :py:class:`~ibm_watsonx_data_integration.cpd_models.service_model.Service` object you want to delete into the :py:meth:`Platform.delete_service_instance() <ibm_watsonx_data_integration.platform.Platform.delete_service_instance>` method to delete it.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> service_instance = platform.service_instances.get(name='My Datastage Service Instance')
    >>> platform.delete_service_instance(service_instance)

.. skip: end
