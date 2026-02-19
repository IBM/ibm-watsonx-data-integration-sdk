.. _preparing_data__batch_schemas:

Schemas
=======

For batch flows, a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema` is shared in the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` between two batch stages. A schema defines the structure (columns, data types) of your data.

To create a schema you must first create a link between two batch stages, which is explained in the :ref:`Connecting Batch Stages <preparing_data__batch__connecting_stages>` section.
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

Schemas are populated with fields. To add a field to a schema use the :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` method which requires an ``odbc_type`` and a ``name``.
You can also add other desired parameters to the :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` method to intialize the field with specific properties.

.. code-block:: python

    >>> field1 = schema.add_field(odbc_type = 'CHAR', name = 'COLUMN_1')
    >>> field2 = schema.add_field('CHAR', 'COLUMN_2', length = 100, nullable = True)

.. _preparing_data__batch_schemas__editing_and_removing_a_schema_field:

Editing and Removing a Schema Field
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To edit an already existing schema field you can directly edit the property you wish to change. To get a list of all properties check the parameters of the :py:meth:`Schema.add_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.add_field>` function. Some properties are dependent on ``odbc_type`` and may not always be settable.

To remove a field simply use the :py:meth:`Schema.remove_field() <ibm_watsonx_data_integration.services.datastage.models.schema.schema.Schema.remove_field>` method which takes in the name of the field.

.. code-block:: python

    >>> field3 = schema.add_field(odbc_type = 'CHAR', name = 'COLUMN_3', length = 100, nullable = True)
    >>> field3.nullable = False
    >>> schema = schema.remove_field('COLUMN_3')

Schemas dictate what data is sent between stages. In this example, the schema has two ``CHAR`` fields called ``COLUMN_1`` and ``COLUMN_2``. As a result, the Row Generator outputs two ``CHAR`` columns which are sent to the Peek stage.

All schemas use data fields for the input and output. Each field is defined by a datatype and name, as seen above. The supported datatypes are as follows:

.. code-block:: python

    'BIGINT', 'BINARY', 'BIT', 'CHAR', 'DATE', 'DECIMAL', 'DOUBLE', 'FLOAT', 'INTEGER',
    'LONGVARBINARY', 'LONGVARCHAR', 'NUMERIC', 'REAL', 'SMALLINT', 'TIME', 'TIMESTAMP',
    'TINYINT', 'UNKNOWN', 'VARBINARY', 'VARCHAR', 'NCHAR', 'LONGNVARCHAR', 'NVARCHAR'

.. note::
    Certain stages (e.g. Copy, Match Frequency) will automatically populate certain schemas in the UI.
    That functionality is replicated in the SDK.
    You should only specify the schemas that you would normally need to specify.

.. _preparing_data__batch_schemas__retreiving_links_and_schemas:

Retreiving existing Links and Schemas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the SDK, if you wish to retreive an existing schema you must first retreive the link the schema is attached to. One way to do this is to use the :py:attr:`BatchFlow.links <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.links>` property to create a :py:class:`list` of all the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` instances in the flow.

.. code-block:: python

    >>> batch_flow.links
    [Link_1 (src='Row_Generator', dest='Peek'), Link_2 (src='Row_Generator_2', dest='Peek_2')]

If you already know the ``name`` of the link you wish to retrieve simply pass the ``name`` into the :py:meth:`BatchFlow.links.get() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.links.get>` method. This method will return a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` instance.

.. code-block:: python

    >>> Link = batch_flow.links.get(name='Link_1')
    >>> Link
    Link_1 (src='Row_Generator', dest='Peek')
