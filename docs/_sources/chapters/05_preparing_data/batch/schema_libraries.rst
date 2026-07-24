.. _preparing_data__schema_libraries:

XML Schema Libraries
====================

A :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema_library.SchemaLibrary` is a project-level asset that holds one or more XSD or JSON schema files defining XML document structures. Schema libraries are used by the Hierarchical Data stage to parse and validate XML data.

A library is created once and can be referenced by multiple Hierarchical Data stages across many flows within the same project.

.. _preparing_data__schema_libraries__creating:

Creating a Schema Library
~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a schema library, use the :py:meth:`Project.create_schema_library() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_schema_library>` method:

.. code-block:: python

    >>> schema_lib = project.create_schema_library(
    ...     name='plant_collection',
    ...     description='XSD for plant collection XML documents'
    ... )

.. _preparing_data__schema_libraries__uploading:

Uploading a Schema File
~~~~~~~~~~~~~~~~~~~~~~~~

After creating the library, upload an XSD or JSON schema file using :py:meth:`SchemaLibrary.upload_schema_file() <ibm_watsonx_data_integration.services.datastage.models.schema.schema_library.SchemaLibrary.upload_schema_file>`. Supported extensions are ``.xsd``, ``.json``, ``.jsn``, and ``.jsd``.

.. skip: start 'local file not available in CI'

.. code-block:: python

    >>> from pathlib import Path
    >>> response = schema_lib.upload_schema_file(
    ...     project=project,
    ...     file=Path('plant_collection.xsd')
    ... )

.. skip: end

The file is automatically wrapped in a zip archive before being sent to the platform. After upload, the library's ``parse_status`` field reflects whether the schema was parsed successfully.

.. _preparing_data__schema_libraries__attaching:

Attaching a Schema Library to a Stage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use a schema library in a Hierarchical Data stage, attach it using :py:meth:`HierarchicalDataStage.add_schema_library() <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.add_schema_library>`:

.. skip: start 'requires a flow not created in this doc'

.. code-block:: python

    >>> flow = project.create_flow(name='xml_processing_flow', flow_type='batch')
    >>> stage = flow.add_stage('Hierarchical Data', 'hierarchical_data')
    >>> stage.add_schema_library(schema_lib)

.. skip: end

Duplicate entries are silently ignored — adding the same library twice has no effect. To detach a library, use :py:meth:`HierarchicalDataStage.remove_schema_library() <ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage.remove_schema_library>`:

.. skip: start 'requires a flow not created in this doc'

.. code-block:: python

    >>> stage.remove_schema_library(schema_lib)

.. skip: end

After modifying the stage, persist the flow:

.. skip: start 'requires a flow not created in this doc'

.. code-block:: python

    >>> project.update_flow(flow)
    <Response [200]>

.. skip: end

.. _preparing_data__schema_libraries__retrieving:

Retrieving Schema Libraries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list all schema libraries in a project, use the :py:attr:`Project.schema_libraries <ibm_watsonx_data_integration.cpd_models.project_model.Project.schema_libraries>` property, which returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema_library.SchemaLibraries` collection:

.. skip: start 'non-deterministic output'

.. code-block:: python

    >>> project.schema_libraries
    [SchemaLibrary(library_id='...', name='plant_collection')]

.. skip: end

.. _preparing_data__schema_libraries__deleting:

Deleting a Schema Library
~~~~~~~~~~~~~~~~~~~~~~~~~~

To delete a schema library, use the :py:meth:`Project.delete_schema_library() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_schema_library>` method:

.. code-block:: python

    >>> project.delete_schema_library(schema_lib)
    <Response [204]>

.. note::
    Schema libraries are project-level assets, independent of any flow. Deleting a library does not automatically remove references to it from existing Hierarchical Data stages.

.. seealso::
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema_library.SchemaLibrary` - API reference for SchemaLibrary class
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.stages.hierarchical_data.HierarchicalDataStage` - API reference for HierarchicalDataStage class
