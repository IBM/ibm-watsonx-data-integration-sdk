.. _getting_started_and_tutorials__platform:

Platform
========


To interact with the watsonx.data integration platform, instantiate the :py:class:`ibm_watsonx_data_integration.platform.Platform` class.

The ``Platform`` object serves as the entry point for accessing all major functionality in the SDK, including service instances, environments, flows, and more.

You must provide a valid authentication object (such as an instance of :py:class:`ibm_watsonx_data_integration.common.auth.IAMAuthenticator`) when initializing the platform object.

Note: If you don’t specify base_api_url, the SDK defaults to the Dallas (us‑south) endpoint at "https://api.dataplatform.cloud.ibm.com". For any other region, set base_api_url to the appropriate regional endpoint.

Examples

Dallas (us-south)
~~~~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth)

Once you’ve created the ``platform`` object, you can begin interacting with the watsonx.data integration platform.

London (eu-gb)
~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth, base_api_url="https://api.eu-gb.dataplatform.cloud.ibm.com")

Tokyo (jp-tok)
~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth, base_api_url="https://api.jp-tok.dataplatform.cloud.ibm.com")

Frankfurt (eu-de)
~~~~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth, base_api_url="https://api.eu-de.dataplatform.cloud.ibm.com")

Sydney (au-syd)
~~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth, base_api_url="https://api.au-syd.dai.cloud.ibm.com")

Toronto (ca-tor)
~~~~~~~~~~~~~~~~
.. code-block:: python

 >>> from ibm_watsonx_data_integration.platform import Platform
 >>> platform = Platform(auth, base_api_url="https://api.ca-tor.dai.cloud.ibm.com")
