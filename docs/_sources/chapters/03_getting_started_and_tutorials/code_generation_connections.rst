.. _getting_started_and_tutorials__code_generation_connections:

Code Generation for Connections
===============================

For connections, you can use the
:py:class:`ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to automate the transformation.

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
:py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.Connection`
class fetched directly from a project.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.codegen import PythonGenerator
    >>>
    >>> connection_name
    'db2 connection'
    >>>
    >>> connection = project.connections.get(name=connection_name)
    >>> connection
    Connection(name='db2 connection')
    >>>
    >>> generator = PythonGenerator(
    ...     source=connection,
    ...     destination='/tmp/output.py',
    ...     auth=auth,  # pragma: allowlist secret
    ...     base_api_url='https://api.ca-tor.dai.cloud.ibm.com'
    ... )
    >>> generator.save()
    PosixPath('/tmp/output.py')


Example Output
~~~~~~~~~~~~~~

The following is an example of the output saved to ``/tmp/output.py``

.. skip: start

.. code-block:: python

    import os
    from ibm_watsonx_data_integration import Platform
    from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
    auth = IAMAuthenticator( # pragma: allowlist secret
        api_key=os.getenv('IBM_CLOUD_API_KEY'),
        base_auth_url='https://cloud.ibm.com',
    )
    platform = Platform(
        auth=auth, # pragma: allowlist secret
        base_url='https://cloud.ibm.com',
        base_api_url='https://api.ca-tor.dai.cloud.ibm.com',
    )
    project = platform.projects.get(project_id='29c07359-c05b-4868-aa95-381e03d568e0')
    connection = project.create_connection(
        name='db2-conn',
        datasource_type=platform.datasources.get(datasource_id='8c1a4480-1c29-4b33-9086-9cb799d7b157'),
        description='',
        properties={
            'database': 'mydatabase',
            'password': 'mypassword', # pragma: allowlist secret
            'port': '3000',
            'username_password_security': 'default',
            'host': 'myhost',
            'ssl': 'false',
            'username_password_encryption': 'default',
            'username': 'myusername'
        }
    )

.. skip: end
