.. _getting_started_and_tutorials__authentication:

Authentication
==============

To interact with the watsonx.data integration platform via the SDK, you must first authenticate.
The SDK supports multiple authentication methods.

IAM Authentication (SaaS)
-------------------------

You can generate an API key from your IBM Cloud account at: `https://cloud.ibm.com/iam/apikeys <https://cloud.ibm.com/iam/apikeys>`_

To authenticate, pass your API key into the :py:class:`ibm_watsonx_data_integration.common.auth.IAMAuthenticator` class.

.. skip: start 'dummy authentication values'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
    >>>
    >>> # Replace with your actual API key
    >>> auth = IAMAuthenticator(                       # pragma: allowlist secret
    ...     api_key='your-api-key',                    # pragma: allowlist secret
    ...     base_auth_url='https://cloud.ibm.com'
    ... )

.. skip: end

.. note::
   The ``base_auth_url`` should be the host URL where you created your IAM API key.
   In the example above, the ``base_auth_url`` is set to ``'https://cloud.ibm.com'`` since the API key was created at `'https://cloud.ibm.com/iam/apikeys'`.
   In addition, the ``base_auth_url`` is an optional parameter and will default to ``'https://cloud.ibm.com'``.


Bearer Token Authentication (SaaS)
----------------------------------

Alternatively, you can authenticate using a bearer token by passing it into the :py:class:`ibm_watsonx_data_integration.common.auth.BearerTokenAuthenticator` class.

For more information on how to generate bearer tokens, please refer to `this page <https://cloud.ibm.com/docs/account?topic=account-iamtoken_from_apikey>`_.

.. skip: start 'dummy authentication values'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.common.auth import BearerTokenAuthenticator
    >>>
    >>> # Replace with your actual bearer token
    >>> auth = BearerTokenAuthenticator(               # pragma: allowlist secret
    ...     bearer_token='your-bearer-token'           # pragma: allowlist secret
    ... )

.. skip: end

Zen Api Key Authentication (on-prem)
------------------------------------
For on-premises authentication, use the :py:class:`ibm_watsonx_data_integration.common.auth.ZenApiKeyAuthenticator` class.
Pass in your username and Zen API key into the ``username`` and ``zen_api_key`` fields respectively:

.. skip: start 'dummy authentication values'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.common.auth import ZenApiKeyAuthenticator
    >>>
    >>> # Replace with your actual username and zen_api_key
    >>> auth = ZenApiKeyAuthenticator(               # pragma: allowlist secret
    ...     username='your-username',                # pragma: allowlist secret
    ...     zen_api_key='your-zen-api-key'           # pragma: allowlist secret
    ... )

.. skip: end

ICP4D Authentication (on-prem)
------------------------------

For on-premises authentication, you can also use the :py:class:`ibm_watsonx_data_integration.common.auth.ICP4DAuthenticator` class.
Pass your username and cluster URL to the ``username`` and ``url`` fields, respectively.
Finally, you may authenticate using either a traditional password or your Zen API key by using the ``password`` or ``zen_api_key`` fields, respectively. Only one of ``password`` or ``zen_api_key`` is required.

.. skip: start 'dummy authentication values'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.common.auth import ICP4DAuthenticator
    >>>
    >>> # Replace with your actual username and password
    >>> auth = ICP4DAuthenticator(                   # pragma: allowlist secret
    ...     url='your-cluster-url',                  # pragma: allowlist secret
    ...     username='your-username',                # pragma: allowlist secret
    ...     password='your-password'                 # pragma: allowlist secret
    ...     # zen_api_key='your-zen-api-key'         # pragma: allowlist secret
    ... )

.. skip: end

Using the Authenticator
-----------------------

Once authenticated, pass the ``auth`` object to the :ref:`Platform class <getting_started_and_tutorials__platform>` to begin interacting with the SDK.

Regional Endpoints (SaaS)
--------------------------

When initializing the :py:class:`~ibm_watsonx_data_integration.platform.Platform`, you can specify a regional endpoint via the ``base_api_url`` parameter.
If not specified, the SDK defaults to the Dallas endpoint.

The SDK provides an :py:class:`~ibm_watsonx_data_integration.common.constants.IBMCloudRegion` enum for convenient access to regional endpoints:

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Region
     - Enum Value
     - URL
   * - Dallas
     - ``IBMCloudRegion.DALLAS``
     - ``https://api.dataplatform.cloud.ibm.com``
   * - Frankfurt
     - ``IBMCloudRegion.FRANKFURT``
     - ``https://api.eu-de.dataplatform.cloud.ibm.com``
   * - London
     - ``IBMCloudRegion.LONDON``
     - ``https://api.eu-gb.dataplatform.cloud.ibm.com``
   * - Tokyo
     - ``IBMCloudRegion.TOKYO``
     - ``https://api.jp-tok.dataplatform.cloud.ibm.com``
   * - Sydney
     - ``IBMCloudRegion.SYDNEY``
     - ``https://api.au-syd.dai.cloud.ibm.com``
   * - Toronto
     - ``IBMCloudRegion.TORONTO``
     - ``https://api.ca-tor.dai.cloud.ibm.com``

.. skip: start 'production url usage'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.platform import Platform
    >>> from ibm_watsonx_data_integration.common.constants import IBMCloudRegion
    >>>
    >>> # Connect to the Frankfurt regional endpoint using the enum (recommended)
    >>> platform = Platform(auth, base_api_url=IBMCloudRegion.FRANKFURT)
    >>>
    >>> # Alternatively, you can still use the URL string directly
    >>> platform = Platform(auth, base_api_url='https://api.eu-de.dataplatform.cloud.ibm.com')

.. skip: end

For a full list of available endpoints, refer to the `IBM Cloud API documentation <https://cloud.ibm.com/apidocs/data-ai-common-core>`_.
