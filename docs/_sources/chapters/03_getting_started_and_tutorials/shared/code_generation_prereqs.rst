Prerequisites
~~~~~~~~~~~~~

To properly generate a script, you will need a correctly configured authenticator class (for example: :py:class:`~ibm_watsonx_data_integration.common.auth.IAMAuthenticator`).

Masking Authentication Credentials
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The part of the generated script will include the creation of an authenticator object based on the one you have configured.
By default, the sensitive values, such as api keys, will be replaced with environment variables. If you would like to include
sensitive credentials in your script, you can pass ``mask_credentials=False`` into the constructor of
:py:class:`ibm_watsonx_data_integration.codegen.generator.PythonGenerator`.

If you do choose to leave the credentials masked, you can expect the following environment variables to replace their
corresponding credentials:

* **IBM_CLOUD_API_KEY**: :py:attr:`IAMAuthenticator.api_key <ibm_watsonx_data_integration.common.auth.IAMAuthenticator.api_key>`
* **ZEN_API_KEY**: :py:attr:`ZenApiKeyAuthenticator.zen_api_key <ibm_watsonx_data_integration.common.auth.ZenApiKeyAuthenticator.zen_api_key>` and :py:attr:`ICP4DAuthenticator.zen_api_key <ibm_watsonx_data_integration.common.auth.ICP4DAuthenticator.zen_api_key>`
* **CP4D_PASSWORD**: :py:attr:`ICP4DAuthenticator.password <ibm_watsonx_data_integration.common.auth.ICP4DAuthenticator.password>`
* **BEARER_TOKEN**: :py:attr:`BearerTokenAuthenticator.bearer_token <ibm_watsonx_data_integration.common.auth.BearerTokenAuthenticator.bearer_token>`
