.. _getting_started_and_tutorials__python_code_generation:

Python Code Generation
======================

If you already have a set of existing flows and want to recreate them into Python SDK, you can use the
:py:class:`~ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to automate the transformation.

Prerequisites
~~~~~~~~~~~~~

To properly generate a script, a few preparatory steps are required:

1. Create an instance of a correctly configured and authenticated :py:class:`~ibm_watsonx_data_integration.common.auth.IAMAuthenticator` class.
2. Make sure your IBM Cloud API key is exported as an environment variable named ``IBM_CLOUD_API_KEY``. This key will be used by the generated script during execution.

Example usage
~~~~~~~~~~~~~

The example below demonstrates how to regenerate Python SDK code from
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamsetsFlow`
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
