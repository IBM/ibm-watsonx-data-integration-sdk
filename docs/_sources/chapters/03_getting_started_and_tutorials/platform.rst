.. _getting_started_and_tutorials__platform:

Platform
========

To interact with the watsonx.data integration platform, instantiate the :py:class:`ibm_watsonx_data_integration.platform.Platform` class.

The ``Platform`` object serves as the entry point for accessing all major functionality in the SDK, including service instances, environments, flows, and more.

You must provide a valid authentication object (such as an instance of :py:class:`ibm_watsonx_data_integration.common.auth.IAMAuthenticator`) when initializing the platform object.

.. note::
    If you don’t specify `base_api_url`, the SDK defaults to `'https://api.dataplatform.cloud.ibm.com'`. For any other region, set base_api_url to the appropriate regional endpoint.

.. skip: start 'production url usage'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.platform import Platform
    >>> platform = Platform(auth, base_api_url='https://api.ca-tor.dai.cloud.ibm.com')

.. skip: end

Once you’ve created the ``Platform`` object, you can begin interacting with the watsonx.data integration platform.


.. note::
    When connecting to an on-premises deployment, specify your cluster’s URL using the `base_url` parameter and provide your authenticator object through the `auth` parameter as usual.
