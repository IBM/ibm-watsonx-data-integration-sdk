.. _administration__accounts:

Accounts
========

|

An account in IBM Cloud is the top-level container for resource grouping, access control, and billing.
It provides the context under which all Cloud Pak Data integration workloads—projects, jobs, flows, and connections—operate.
The SDK provides functionality to interact with the accounts on the watsonx.data integration platform.

This includes operations such as:
    * Listing all accessible accounts
    * Fetching an account by ID
    * Getting the current account
    * Setting the current account

Listing all accessible accounts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, click your account name in the top-right corner to open the account picker and view all accounts you have access to.

.. image:: ../../_static/images/accounts/list_accounts.png
   :alt: Screenshot of the Account lists in the UI
   :align: center
   :width: 100%

Accounts can be retrieved using :py:attr:`Platform.accounts <ibm_watsonx_data_integration.platform.Platform.accounts>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.account_model.Accounts` object.

.. code-block:: python

    >>> all_accounts = platform.accounts
    >>> all_accounts
    [
        Account(name="A's Account", account_type='TRIAL', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxa'),
        Account(name="B's Account", account_type='TRIAL', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9'),
        Account(name="C's Account", account_type='PAYG', account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6'),
        Account(name="D's Account", account_type='PAYG', account_id='fxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9'),
        Account(name="E's Account", account_type='PAYG', account_id='9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxc')
    ]

Fetching an account by ID
~~~~~~~~~~~~~~~~~~~~~~~~~

Accounts can be retrieved using :py:attr:`Platform.accounts <ibm_watsonx_data_integration.platform.Platform.accounts>` property.
You can also further filter and refine the account returned based on the ``account_id`` attribute.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.account_model.Account` object.

.. code-block:: python

    >>> account = platform.accounts.get(account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6')
    >>> account
    Account(name="C's Account", account_type='PAYG', account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6')

Getting the current account
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can use the :py:attr:`Platform.current_account <ibm_watsonx_data_integration.platform.Platform.current_account>` property to retrieve the account that’s currently in scope for all SDK operations.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.account_model.Account` object.
By default, it is the first account you joined or the first one listed in your account list.

.. code-block:: python

    >>> account = platform.current_account
    >>> account
    Account(name="A's Account", account_type='TRIAL', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxa')

Setting the current account
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can use the setter for the :py:attr:`Platform.current_account <ibm_watsonx_data_integration.platform.Platform.current_account>` to override which account will be used for all subsequent SDK operations.
Pass in an :py:class:`~ibm_watsonx_data_integration.cpd_models.account_model.Account` object.

.. code-block:: python

    >>> all_accounts = platform.accounts
    >>> all_accounts
    [
        Account(name="A's Account", account_type='TRIAL', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxa'),
        Account(name="B's Account", account_type='TRIAL', account_id='6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9'),
        Account(name="C's Account", account_type='PAYG', account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6'),
        Account(name="D's Account", account_type='PAYG', account_id='fxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx9'),
        Account(name="E's Account", account_type='PAYG', account_id='9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxc')
    ]
    >>> account = platform.accounts.get(account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6')
    >>> platform.current_account = account
    >>> platform.current_account
    Account(name="C's Account", account_type='PAYG', account_id='bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6')
