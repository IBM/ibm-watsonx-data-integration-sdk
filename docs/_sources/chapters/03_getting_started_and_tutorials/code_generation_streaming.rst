.. _getting_started_and_tutorials__code_generation_streaming:

Code Generation (Streaming)
===========================

:py:class:`ibm_watsonx_data_integration.codegen.generator.PythonGenerator` is an entry point for converting existing flows to Python SDK code.

For streaming flows, you can use the
:py:class:`~ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to automate the transformation.

.. include:: ./shared/code_generation_prereqs.rst

Example usage
~~~~~~~~~~~~~

The example below demonstrates how to regenerate Python SDK code from a
:py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow`
object fetched directly from a project.

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

Generating Jupyter Notebooks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also generate code as a Jupyter notebook instead of a Python script. This is useful for interactive development
and documentation. The notebook organizes the code into logical cells:

- **Imports**: All required imports
- **Authentication**: Setting up the authenticator
- **Platform Setup**: Creating the Platform instance
- **Project Setup**: Getting or creating the project
- **Environment Setup**: Setting up the streaming environment
- **Flow Creation**: Creating the flow
- **Stage Creation**: Creating all stages
- **Stage Connections**: Connecting stages together
- **Flow Update**: Updating the flow with the complete DAG

.. skip: start 'todo setup proper python generator doc test'

.. code-block:: python

    >>> from ibm_watsonx_data_integration.codegen.generator import PythonGenerator, PythonGeneratorFormat
    >>>
    >>> # Generate as Jupyter notebook
    >>> generator = PythonGenerator(
    ...     source=flow,
    ...     destination='/tmp/output.ipynb',
    ...     auth=auth,  # pragma: allowlist secret
    ...     base_api_url='https://api.ca-tor.dai.cloud.ibm.com',
    ...     output_format=PythonGeneratorFormat.NOTEBOOK
    ... )
    >>> generator.save()
    PosixPath('/tmp/output.ipynb')

.. skip: end

The generated notebook can be opened in JupyterLab, Jupyter Notebook, or any compatible notebook environment.
