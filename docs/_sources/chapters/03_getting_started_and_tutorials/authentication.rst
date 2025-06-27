.. _getting_started_and_tutorials__authentication:

Authentication
==============

To interact with the watsonx.data integration platform via the SDK, you must first authenticate.
The SDK supports IAM authentication using an API key.

You can generate an API key from your IBM Cloud account at: `https://cloud.ibm.com/iam/apikeys <https://cloud.ibm.com/iam/apikeys>`_

To authenticate, pass your API key into the :py:class:`ibm_watsonx_data_integration.common.auth.IAMAuthenticator` class.

.. code-block:: python

 >>> from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
 >>>
 >>> # Replace with your actual API key
 >>> auth = IAMAuthenticator(                       # pragma: allowlist secret
 ...     api_key="your-api-key",                    # pragma: allowlist secret
 ...     base_auth_url="https://cloud.ibm.com"
 ... )

.. note::
   The ``base_auth_url`` should be the host URL where you created your IAM API key.
   In the example above, the ``base_auth_url`` is set to ``"https://cloud.ibm.com"`` since the API key was created at `"https://cloud.ibm.com/iam/apikeys"`.

Once authenticated, pass the ``auth`` object to the :ref:`Platform class<getting_started_and_tutorials__platform>` to begin interacting with the SDK.
