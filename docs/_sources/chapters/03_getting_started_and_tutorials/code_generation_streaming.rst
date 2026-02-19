.. _getting_started_and_tutorials__code_generation_streaming:

Code Generation (Streaming)
===========================

:py:class:`ibm_watsonx_data_integration.codegen.generator.PythonGenerator` is an entry point that converts existing flows to Python SDK code.

For streaming flows, you can use the
:py:class:`~ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to automate the transformation.

.. include:: ./shared/code_generation_prereqs.rst

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
