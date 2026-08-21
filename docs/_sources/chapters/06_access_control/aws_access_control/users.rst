.. _access_control__users_aws:

Users
=====

Users include details such as user ID, email, display name, and status.
User Management is a platform-level service in AWS deployment that enables you to manage the users within your account.
The SDK provides functionality to interact with the User Management API on the AWS watsonx.data integration platform.

This includes operations such as:
    * Listing all users
    * Getting user details using the user ID

Listing all users
~~~~~~~~~~~~~~~~~

Users can be retrieved using the :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model_aws.UserProfilesAWS` object.

.. skip: start 'AWS Users are only available on aws'

.. code-block:: python

    >>> platform.users
    [...UserProfileAWS(...)...]

.. skip: end

Getting user details using the user ID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Users can be retrieved using the :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
You can also filter the returned users using the ``uid`` attribute.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model_aws.UserProfileAWS` object.

.. skip: start 'unable to create test user'

.. code-block:: python

    >>> platform.users.get(uid='user-uid-here')
    UserProfileAWS(...)

.. skip: end
