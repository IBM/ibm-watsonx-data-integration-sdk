.. _getting_started_and_tutorials__code_generation_streaming:

Code Generation (Streaming)
===========================

:py:class:`ibm_watsonx_data_integration.codegen.generator.PythonGenerator` is an entry point that converts existing flows to Python SDK code.

For streaming flows, you can use the
:py:class:`~ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to automate the transformation.

Prerequisites
~~~~~~~~~~~~~

To properly generate a script, you will need a correctly configured authenticator class (for example: :py:class:`~ibm_watsonx_data_integration.common.auth.IAMAuthenticator`).
If you are using the :py:class:`~ibm_watsonx_data_integration.common.auth.IAMAuthenticator`, make sure your IBM Cloud API key is exported as an environment variable
named ``IBM_CLOUD_API_KEY``. This key will be used by the generated script during execution.

.. note::
   All types of authenticator objects are accepted by the PythonGenerator class, but for now the authenticator object in the generated code will always be IAMAuthenticator.

Example usage
~~~~~~~~~~~~~

The example below demonstrates how to regenerate Python SDK code from
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow`
class fetched directly from a project.

.. skip: start 'todo setup proper python generator doc test'

.. code-block:: python

    >>> import os
    >>> from ibm_watsonx_data_integration.platform import Platform
    >>> from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
    >>> from ibm_watsonx_data_integration.codegen import PythonGenerator
    >>>
    >>> auth = IAMAuthenticator(api_key=os.getenv('IBM_CLOUD_API_KEY'))
    >>> platform = Platform(auth, base_api_url='https://api.ca-tor.dai.cloud.ibm.com')
    >>> project = platform.projects.get(name='Test Project')
    >>> flow = project.flows.get(name='My first flow')
    >>>
    >>> generator = PythonGenerator(
    ...     source=flow,
    ...     destination='/tmp/output.py',
    ...     auth=auth,  # pragma: allowlist secret
    ...     base_api_url='https://api.ca-tor.dai.cloud.ibm.com'
    ... )
    >>> generator.save()
    PosixPath('/tmp/output.py')

.. skip: end
