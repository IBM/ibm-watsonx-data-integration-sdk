.. _preparing_data__data_definitions:

Data Definitions
================

A :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition` is a reusable table definition that can be shared across multiple batch flows. Data definitions define the structure of your data, including its columns and data types, and can be imported into schemas or exported from schemas for reuse.

Data definitions are particularly useful when you have a common data structure that needs to be used in multiple flows, because they allow you to define the structure once and reuse it everywhere.

.. _preparing_data__data_definitions__creating:

Creating a Data Definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a data definition, use the :py:meth:`Project.create_data_definition() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_data_definition>` method. You can create an empty data definition and add fields later, or provide fields during creation.

.. code-block:: python

    >>> data_def = project.create_data_definition(
    ...     name='Customer Data',
    ...     description='Customer information schema'
    ... )

.. _preparing_data__data_definitions__adding_fields:

Adding Fields to a Data Definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Data definitions are populated with fields, similar to schemas. To add a field to a data definition, use the :py:meth:`DataDefinition.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition.add_field>` method, which requires an ``odbc_type`` and a ``name``.

.. code-block:: python

    >>> field1 = data_def.add_field(odbc_type='VARCHAR', name='CUSTOMER_ID')
    >>> field2 = data_def.add_field('CHAR', 'CUSTOMER_NAME', length=100)
    >>> field3 = data_def.add_field('INTEGER', 'AGE')

After adding fields, you must update the data definition to persist the changes:

.. code-block:: python

    >>> project.update_data_definition(data_def)
    <Response [200]>

The supported datatypes are the same as for schemas:

.. code-block:: python

    'BIGINT', 'BINARY', 'BIT', 'CHAR', 'DATE', 'DECIMAL', 'DOUBLE', 'FLOAT', 'INTEGER',
    'LONGVARBINARY', 'LONGVARCHAR', 'NUMERIC', 'REAL', 'SMALLINT', 'TIME', 'TIMESTAMP',
    'TINYINT', 'UNKNOWN', 'VARBINARY', 'VARCHAR', 'NCHAR', 'LONGNVARCHAR', 'NVARCHAR'

.. _preparing_data__data_definitions__editing_and_removing_fields:

Editing and Removing Fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To edit an existing field in a data definition, you can directly modify the field properties. For a list of all properties, see the parameters of the :py:meth:`DataDefinition.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition.add_field>` method. Some properties depend on ``odbc_type`` and may not always be settable.

To remove a field, use the :py:meth:`DataDefinition.remove_field() <ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition.remove_field>` method, which takes the field name.

.. code-block:: python

    >>> field4 = data_def.add_field('CHAR', 'TEMP_FIELD', length=50)
    >>> field4.configuration.nullable = False
    >>> data_def.remove_field('TEMP_FIELD')
    True
    >>> project.update_data_definition(data_def)
    <Response [200]>

To remove all fields at once, use the :py:meth:`DataDefinition.clear_fields() <ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition.clear_fields>` method:

.. invisible-code-block: python

    >>> # Store the original fields
    >>> from copy import deepcopy
    >>> original_fields = deepcopy(data_def.fields)

.. code-block:: python

    >>> data_def.clear_fields()
    >>> project.update_data_definition(data_def)
    <Response [200]>

.. invisible-code-block: python

    >>> # Restore the original fields
    >>> data_def.fields = [field.clone() for field in original_fields]
    >>> project.update_data_definition(data_def)
    <Response [200]>

.. _preparing_data__data_definitions__retrieving:

Retrieving Data Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To retrieve existing data definitions from a project, use the :py:attr:`Project.data_definitions <ibm_watsonx_data_integration.cpd_models.project_model.Project.data_definitions>` property which returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinitions` collection.

.. code-block:: python

    >>> project.data_definitions
    [DataDefinition(metadata=DataDefinitionMetadata(name='Customer Data', description='Customer information schema'))]

If you know the ``data_definition_id``, you can retrieve a specific data definition using the :py:meth:`DataDefinitions.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method:

.. skip: next

.. code-block:: python

    >>> data_def = project.data_definitions.get(data_definition_id='abc-123-def')


.. _preparing_data__data_definitions__importing_to_schema:

Importing Data Definition into a Schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

One of the key benefits of data definitions is the ability to import them into schemas. This allows you to reuse a common data structure across multiple flows. Use the :py:meth:`Schema.import_data_definition() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.import_data_definition>` method to import fields from a data definition into a schema.

.. code-block:: python

    >>> row_gen = batch_flow.add_stage('Row Generator', 'Row_Generator_1')
    >>> peek = batch_flow.add_stage('Peek', 'Peek_1')
    >>> link = row_gen.connect_output_to(peek)
    >>> schema = link.create_schema()
    >>>
    >>> # Import all fields from the data definition
    >>> schema.import_data_definition(data_def)
    >>> schema.fields
    [VarChar(name=CUSTOMER_ID, type=VARCHAR), Char(name=CUSTOMER_NAME, type=CHAR), Integer(name=AGE, type=INTEGER)]

You can also import only specific fields by providing a list of field names to the ``field_names`` parameter:

.. invisible-code-block: python
    >>> schema.fields = []

.. code-block:: python

    >>> # Import only specific fields
    >>> schema.import_data_definition(data_def, field_names=['CUSTOMER_ID', 'CUSTOMER_NAME'])
    >>> schema.fields
    [VarChar(name=CUSTOMER_ID, type=VARCHAR), Char(name=CUSTOMER_NAME, type=CHAR)]

Fields are always appended to the existing schema, so you can import from multiple data definitions or add additional fields manually.

.. _preparing_data__data_definitions__exporting_from_schema:

Exporting Schema to Data Definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also create a data definition from an existing schema using the :py:meth:`Schema.export_data_definition() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.export_data_definition>` method. This is useful when you have designed a schema in a flow and want to reuse it elsewhere.

.. code-block:: python

    >>> # Create a data definition from an existing schema
    >>> new_data_def = schema.export_data_definition(
    ...     name='Exported Schema',
    ...     description='Schema exported from batch flow'
    ... )
    >>> new_data_def.fields
    [VarChar(name=CUSTOMER_ID, type=VARCHAR), Char(name=CUSTOMER_NAME, type=CHAR)]

The exported data definition will contain all fields from the schema and will be automatically created in the project.

To specify a selection of fields to export, pass the field names to the ``field_names`` parameter.

.. code-block:: python

    >>> new_data_def = schema.export_data_definition(
    ...     name='Exported Schema with only customer_id',
    ...     description='Schema exported from batch flow',
    ...     field_names=['CUSTOMER_ID']
    ... )
    >>> new_data_def.fields
    [VarChar(name=CUSTOMER_ID, type=VARCHAR)]

.. _preparing_data__data_definitions__duplicating:

Duplicating Data Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a copy of an existing data definition, use the :py:meth:`Project.duplicate_data_definition() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_data_definition>` method:

.. code-block:: python

    >>> duplicated = project.duplicate_data_definition(
    ...     data_definition=data_def,
    ...     name='Customer Data Copy'
    ... )

If you do not provide a name, the duplicated data definition will be named ``{{original name}} cloned``.

.. _preparing_data__data_definitions__updating_and_deleting:

Updating and Deleting Data Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To update a data definition after making changes, use the :py:meth:`Project.update_data_definition() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_data_definition>` method:

.. code-block:: python

    >>> data_def.name = 'Updated Customer Data'
    >>> data_def.description = 'Updated description'
    >>> project.update_data_definition(data_def)
    <Response [200]>

To delete a data definition, use the :py:meth:`Project.delete_data_definition() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_data_definition>` method:

.. code-block:: python

    >>> project.delete_data_definition(data_def)
    <Response [200]>

.. _preparing_data__data_definitions__working_with_fields:

Working with Fields
~~~~~~~~~~~~~~~~~~~

Data definitions provide several methods for working with fields:

**Getting a field by name:**

.. code-block:: python

    >>> field = data_def.get_field_by_name('CUSTOMER_ID')
    >>> print(field.configuration.odbc_type)
    VARCHAR

**Checking if a field exists:**

.. code-block:: python

    >>> if data_def.has_field('CUSTOMER_ID'):
    ...     print('Field exists')
    Field exists
    >>>
    >>> # Alternative using 'in' operator
    >>> if 'CUSTOMER_ID' in data_def:
    ...     print('Field exists')
    Field exists

**Accessing all fields:**

.. code-block:: python

    >>> for field in data_def.fields:
    ...     print(f'{field.configuration.name}: {field.configuration.odbc_type}')
    CUSTOMER_ID: VARCHAR
    CUSTOMER_NAME: CHAR
    AGE: INTEGER

.. note::
    Data definitions are stored as project assets and can be shared across multiple flows within the same project. Changes to a data definition do not automatically update schemas that have already imported from it; you must re-import to get the updated fields.

.. seealso::
    - :ref:`Schemas <preparing_data__batch_schemas>` - For information about schemas used in batch flows
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition` - API reference for DataDefinition class
    - :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema` - API reference for Schema class
