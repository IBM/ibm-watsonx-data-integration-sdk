.. _preparing_data__batch_schemas:

Schemas
=======

For batch flows, a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema` is shared on the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` between two batch stages. A schema defines the structure of your data, including its columns and data types.

To create a schema, you must first create a link between two batch stages, as explained in the :ref:`Connecting Batch Stages <preparing_data__batch__connecting_stages>` section.
Then use the :py:meth:`Link.create_schema() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link.create_schema>` method to create the schema object.

.. code-block:: python

    >>> row_gen = batch_flow.add_stage('Row Generator', 'Row_Generator_2')
    >>> peek = batch_flow.add_stage('Peek', 'Peek_2')
    >>> link = row_gen.connect_output_to(peek)
    >>> link.name = 'Link_2'
    >>> schema = link.create_schema()

.. _preparing_data__batch_schemas__initializing_a_schema_field:

Initializing a Schema Field
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Schemas are populated with fields. To add a field to a schema, use the :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` method, which requires an ``odbc_type`` and a ``name``.
You can also pass additional parameters to :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` to initialize the field with specific properties.

.. code-block:: python

    >>> field1 = schema.add_field(odbc_type = 'CHAR', name = 'COLUMN_1')
    >>> field2 = schema.add_field('CHAR', 'COLUMN_2', length = 100, nullable = True)

.. _preparing_data__batch_schemas__editing_and_removing_a_schema_field:

Editing and Removing a Schema Field
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To edit an existing schema field, directly modify the property you want to change. For a list of all properties, see the parameters of the :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` method. Some properties depend on ``odbc_type`` and may not always be settable.

To remove a field, use the :py:meth:`Schema.remove_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.remove_field>` method, which takes the field name.

.. code-block:: python

    >>> field3 = schema.add_field(odbc_type = 'CHAR', name = 'COLUMN_3', length = 100, nullable = True)
    >>> field3.nullable = False
    >>> schema = schema.remove_field('COLUMN_3')

Schemas dictate what data is sent between stages. In this example, the schema has two ``CHAR`` fields called ``COLUMN_1`` and ``COLUMN_2``. As a result, the Row Generator outputs two ``CHAR`` columns which are sent to the Peek stage.

All schemas use data fields for input and output. Each field is defined by a data type and a name, as shown above. The supported data types are:

.. code-block:: python

    'BIGINT', 'BINARY', 'BIT', 'CHAR', 'DATE', 'DECIMAL', 'DOUBLE', 'FLOAT', 'INTEGER',
    'LONGVARBINARY', 'LONGVARCHAR', 'NUMERIC', 'REAL', 'SMALLINT', 'TIME', 'TIMESTAMP',
    'TINYINT', 'UNKNOWN', 'VARBINARY', 'VARCHAR', 'NCHAR', 'LONGNVARCHAR', 'NVARCHAR'

.. note::
    Certain stages (e.g. Copy, Match Frequency) will automatically populate certain schemas in the UI.
    That functionality is replicated in the SDK.
    You should only specify the schemas that you would normally need to specify.

.. _preparing_data__batch_schemas__retrieving_links_and_schemas:

Retrieving existing Links and Schemas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the SDK, if you want to retrieve an existing schema, you must first retrieve the link to which the schema is attached. One way to do this is to use the :py:attr:`BatchFlow.links <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.links>` property to get a :py:class:`list` of all :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` instances in the flow.

.. code-block:: python

    >>> batch_flow.links
    [Link_1 (src='Row_Generator', dest='Peek'), Link_2 (src='Row_Generator_2', dest='Peek_2')]

If you already know the ``name`` of the link you want to retrieve, simply pass that ``name`` to the :py:meth:`BatchFlow.links.get() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.links.get>` method. This method returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` instance.

.. code-block:: python

    >>> Link = batch_flow.links.get(name='Link_1')
    >>> Link
    Link_1 (src='Row_Generator', dest='Peek')

.. _preparing_data__batch_schemas__working_with_data_definitions:

Working with Data Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Schemas can import fields from :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.data_definition.DataDefinition` objects, and can also export their fields to create new data definitions. This lets you reuse common data structures across multiple flows.

For detailed information about importing and exporting data definitions with schemas, see:

- :ref:`Importing Data Definition into a Schema <preparing_data__data_definitions__importing_to_schema>` - Import reusable data definitions into schemas
- :ref:`Exporting Schema to Data Definition <preparing_data__data_definitions__exporting_from_schema>` - Create data definitions from existing schemas
- :ref:`Data Definitions <preparing_data__data_definitions>` - Complete guide to working with data definitions
