.. _administration__users_on_prem:

Users
=====

The SDK provides functionality to interact with the users on Cloud Pak for Data.

This includes operations such as:
    * Listing all users


.. _administration__users_on_prem__listing_users:

Listing all users
~~~~~~~~~~~~~~~~~

Users can be retrieved using :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model_on_prem.UserProfilesOnPrem` object.

.. skip: start 'dummy user values'

.. code-block:: python

    >>> platform.users
    [...UserProfileOnPrem(...)...]

.. skip: end
