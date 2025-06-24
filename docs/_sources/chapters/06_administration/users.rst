.. _administration__users:

Users
=====


Users include details such as user ID, first name, last name, email and IAM ID.
User Management is a platform-level service in IBM Cloud that enables you to manage the users within your account.
The SDK provides functionality to interact with the User Management API on the watsonx.data integration platform.

This includes operations such as:
    * Listing all users
    * Getting user details using the IAM ID

Listing all users
~~~~~~~~~~~~~~~~~

In the UI, you can view all users in the current account by navigating to **Top left hamburger menu -> Administration -> Access (IAM) -> Users**.

.. image:: ../../_static/images/users/list_users.png
   :alt: Screenshot of the User list in the UI  - Step 1
   :align: center
   :width: 100%

.. image:: ../../_static/images/users/list_users1.png
   :alt: Screenshot of the User list in the UI  - Step 2
   :align: center
   :width: 100%

Users can be retrieved using :py:attr:`Platform.users <ibm_watsonx_data_integration.platform.Platform.users>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfiles` object.

.. code-block:: python

    >>> all_users = platform.users
    >>> all_users
    [
        UserProfile(id='9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxd', iam_id='IBMid-6xxxxxxxxV', user_id='A.user@ibm.com', state='ACTIVE', email='A.user@ibm.com', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9', added_on='2025-02-18T21:35:47Z'),
        UserProfile(id='1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3', iam_id='IBMid-6xxxxxxxxX', user_id='B.user@ibm.com', state='ACTIVE', email='B.user@ibm.com', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9', added_on='2025-02-18T21:35:50Z'),
        UserProfile(id='cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx5', iam_id='IBMid-6xxxxxxxxK', user_id='C.user@ibm.com', state='PENDING', email='C.user@ibm.com', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9', added_on='2025-02-18T21:35:46Z')
    ]


Getting user details using the IAM ID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To view the user details, you can click on the empty area within the row where the user name is displayed.

.. image:: ../../_static/images/users/get_user_details.png
   :alt: Screenshot of the User details in the UI
   :align: center
   :width: 100%

Pass the ``iam_id` to the :py:meth:`Platform.get_user_profile() <ibm_watsonx_data_integration.platform.Platform.get_user_profile>` method.
This method returns an :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile` object.

.. code-block:: python

    >>> user = platform.get_user_profile('IBMid-6xxxxxxxxV')
    >>> user
    UserProfile(id='9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxd', iam_id='IBMid-6xxxxxxxxV', user_id='A.user@ibm.com', state='ACTIVE', email='A.user@ibm.com', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9', added_on='2025-02-18T21:35:47Z')
