.. _administration__users:

Users
=====


Users include details such as user ID, first name, last name, email and IAM ID.
User Management is a platform-level service in IBM Cloud that enables you to manage the users within your account.
The SDK provides functionality to interact with the User Management API on the watsonx.data integration platform.

This includes operations such as:
    * Listing all users
    * Getting user details using the IAM ID or User ID

Listing all users
~~~~~~~~~~~~~~~~~

In the UI, you can view all users in the current account by navigating to **Top left hamburger menu -> Administration -> Access (IAM) -> Users**.

.. image:: /_static/images/users/list_users.png
   :alt: Screenshot of the User list in the UI  - Step 1
   :align: center
   :width: 100%

.. image:: /_static/images/users/list_users1.png
   :alt: Screenshot of the User list in the UI  - Step 2
   :align: center
   :width: 100%

Users can be retrieved using :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfiles` object.

.. code-block:: python

    >>> platform.users
    [...UserProfile(...)...]

Getting user details using the IAM ID or User ID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To view the user details, you can click on the empty area within the row where the user name is displayed.

.. image:: /_static/images/users/get_user_details.png
   :alt: Screenshot of the User details in the UI
   :align: center
   :width: 100%

Users can be retrieved using :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
You can also further filter the users returned based on the ``iam_id`` or ``user_id`` attribute, but not both.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile` object.

.. skip: start 'unable to create test user'

.. code-block:: python

    >>> platform.users.get(iam_id='IBMid-6xxxxxxxxV')
    UserProfile(...)

    >>> platform.users.get(user_id='B.user@ibm.com')
    UserProfile(...)

.. skip: end
