Preparing Inputs
~~~~~~~~~~~~~~~~

The input can either be a flow JSON or a zip file containing one or more flows and their dependent assets.
Both of these formats can be acquired in numerous ways, but one method is to download it directly from the flow editor.

.. image:: /_static/images/code_generation/sample_project.png
   :alt: Screenshot of the flow editor for a batch flow
   :align: center
   :width: 100%

This is a sample project containing one batch flow and one connection used in the flow.

.. image:: /_static/images/code_generation/sample_batch_flow.png
   :alt: Screenshot of a project containing a sample batch flow and a sample connection
   :align: center
   :width: 100%

The flow reads data from a COS datasource, sorts it, then outputs a preview of the data.

.. image:: /_static/images/code_generation/download_flow.png
   :alt: Screenshot of the popup to download a flow and its dependencies
   :align: center
   :width: 100%

From this page, you can download the flow and its dependencies as a zip file.

This is the expected structure of the downloaded zip file:

.. code-block:: text

   downloaded_folder/
   ├── connections/
   │   └── sample_connection.json
   └── data_intg_flow/
   │   └── sample_batch_flow.json
   └── ...etc

Running the Generator
~~~~~~~~~~~~~~~~~~~~~

Run the PythonGenerator using the :py:meth:`PythonGenerator.generate() <ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator.generate>` method

Inputs:
- ``input_path``: input_path to the JSON or zip file
- ``output_path``: (optional): the file or directory where the code will be written (if no output_path is specified, the code will not be written)
Returns:
- A dictionary that maps file names to the generated code strings.

.. skip: start 'skipping due to fake file paths'

.. code-block:: python

    >>> # Generate multiple flows from zip and write to output directory
    >>> generator.generate(input_path='downloaded_folder.zip', output_path='path/to/output')
    >>> # Generate one flow from JSON and write to output directory
    >>> generator.generate(input_path='downloaded_folder/data_intg_flow/sample_batch_flow.json', output_path='path/to/output.py')
    >>> # Generate from zip without saving files
    >>> generated_code = generator.generate(input_path='downloaded_folder.zip')

.. skip: end

.. note::

   Zip format is preferred since it contains dependencies and therefore can be generated more completely.

Running Generated Code
~~~~~~~~~~~~~~~~~~~~~~

Before running the generated code, three types of placeholders may need to be replaced.

Replace the ``api_key`` and ``project_id`` placeholders if you did not set them on the generator object:

.. code-block:: python

    api_key = '<TODO: insert your api_key>' # pragma: allowlist secret
    project = platform.projects.get(project_id='<TODO: insert your project_id>')

Sensitive or encrypted connection credentials will also be generated as placeholders:

.. code-block:: python

    vertica = project.create_connection(
        name='sample_vertica_conn',
        datasource_type=platform.datasources.get(name='vertica'),
        properties={
            'database': 'abcdef',
            'password': '<TODO: insert your password>',     # pragma: allowlist secret
            'username': '<TODO: insert your username>',
        },
    )
