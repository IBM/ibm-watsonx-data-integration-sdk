.. _access_control__service_ids:

Service IDs
===========

A service ID identifies a service or application similar to how a user ID identifies a user.
Service ID management is a platform level service in IBM Cloud that enables you to manage service IDs under your account.
The SDK provides functionality to interact with the access group API.

.. note::
    The SDK currently only supports retrieving service IDs.

.. _access_control__service_ids__listing_service_ids:

Listing Service IDs
~~~~~~~~~~~~~~~~~~~

In the UI, you can view all service IDs in the current account by **Top left hamburger menu -> Administration -> Access (IAM) -> Service IDs**.

.. image:: /_static/images/service_ids/service_ids.png
   :alt: Screenshot of viewing Service IDs
   :align: center
   :width: 100%


Service IDs can be retrieved using :py:attr:`Platform.service_ids <ibm_watsonx_data_integration.platform.Platform.service_ids>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceIDs` object.

.. code-block:: python

    >>> platform.service_ids.get_all()
    [...ServiceID(...)...]
