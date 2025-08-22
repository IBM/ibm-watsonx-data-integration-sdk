.. _welcome__install:

Prerequisites
=============
.. _prerequisites:

Setup and installation for the ``ibm-watsonx-data-integration`` SDK for Python is lightweight and straight-forward, with only a few dependencies required by the package itself.

For a smooth install and optimal user experience, please be sure to carefully review the prerequisites below prior to installing the SDK and make sure that all requirements have been met.

In order to begin using the SDK, there are a few prerequisites that must be fulfilled:

* The ``ibm-watsonx-data-integration`` package must be :ref:`installed <installation>` on the machine where you plan to execute SDK code.
* A Python 3.10+ interpreter and the pip package manager must both be installed on the machine where the SDK will be used.
  For additional details on the pip package manager as well as installation instructions, please refer to the `pip documentation <https://pip.pypa.io/en/stable/>`_.
* You must have an account with access to the watsonx.data integration platform and an ``IAM API key`` created on that account to be used to :ref:`authenticate<getting_started_and_tutorials__authentication>` from the SDK.
  For additional information on API keys, including how to create and use them, please refer to the `IBM Cloud API Key documentation <https://cloud.ibm.com/docs/account?topic=account-manapikey>`_.
* (Optional but strongly encouraged) For the vast majority of operations on the watsonx.data integration platform, like executing a flow, you will need to generate a ``User API Key`` within the watsonx.data integration service.
  For additional information on User API keys, including how to create and use them, please refer to the `IBM Cloud User API Key documentation <https://cloud.ibm.com/docs/account?topic=account-userapikey>`_.
  Please note that the ``User API key`` is different from the ``IAM API key`` that you use to authenticate in the SDK, mentioned above. The User API key only exists within the watsonx.data integration platform.

  .. note::
     Running StreamSets jobs requires a user API key for secure authorization.
     To verify if your account has an active user API key, click your avatar and select **Profile and settings** to open your account profile.
     Select **User API key** to view the Active keys.
     If you do not have an active API key, create a key by clicking **Create a key**.
