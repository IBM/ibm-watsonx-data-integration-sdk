.. _preparing_data__batch__link_partitioning:

Link Partitioning
=================

Link partitioning controls how data is distributed across parallel processing nodes in DataStage batch flows. The SDK provides methods to configure partitioning on :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` objects between stages.

.. _preparing_data__batch__link_partitioning__setting_partitioning:

Setting Partitioning
~~~~~~~~~~~~~~~~~~~~

To configure partitioning on a link, use the :py:meth:`Link.set_partitioning() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link.set_partitioning>` method. You must specify a ``part_type`` parameter, and can optionally provide ``perform_sort``, ``part_stable``, and ``part_unique`` parameters. This method returns the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` object for method chaining.

Valid ``part_type`` values are: ``'auto'``, ``'hash'``, ``'modulus'``, ``'range'``, ``'roundrobin'``, ``'entire'``, ``'same'``, and ``'random'``.

.. code-block:: python

    >>> batch_flow = project.create_flow(name='Partitioning Example', flow_type='batch')
    >>> source = batch_flow.add_stage('Row Generator', 'Source')
    >>> target = batch_flow.add_stage('Peek', 'Target')
    >>> link = source.connect_output_to(target)
    >>> link.name = 'Link_1'
    >>> link.set_partitioning(part_type='hash', perform_sort=True)
    Link_1 (src='Source', dest='Target')

The ``perform_sort`` parameter is only valid for ``'hash'``, ``'range'``, and ``'modulus'`` partitioning types.

.. note::
    When using modulus partitioning with ``perform_sort=False``, only one partition key is allowed. The SDK automatically sets the ``key_col_select`` attribute to the partition key column name. When ``perform_sort=True``, multiple keys are allowed and ``key_col_select`` is set to ``'default'``.

.. _preparing_data__batch__link_partitioning__adding_partition_keys:

Adding Partition Keys
~~~~~~~~~~~~~~~~~~~~~

After setting the partitioning type, use the :py:meth:`Link.add_partition_key() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link.add_partition_key>` method to specify which columns to use for partitioning. This method requires a ``key_col`` parameter and accepts optional parameters for sorting configuration. This method returns the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` object for method chaining.

.. code-block:: python

    >>> link.add_partition_key('CUSTOMER_ID', sorting=True, sort_order='asc')
    Link_1 (src='Source', dest='Target')
    >>> link.add_partition_key('ORDER_DATE', sorting=True, sort_order='desc')
    Link_1 (src='Source', dest='Target')
    >>> link.add_partition_key('REGION', sorting=False, case_sensitive=False)
    Link_1 (src='Source', dest='Target')

You can also chain these method calls together:

.. invisible-code-block: python

    >>> link.remove_partition_key('CUSTOMER_ID')
    Link_1 (src='Source', dest='Target')
    >>> link.remove_partition_key('ORDER_DATE')
    Link_1 (src='Source', dest='Target')

.. code-block:: python

    >>> link.set_partitioning(part_type='hash', perform_sort=True).add_partition_key('CUSTOMER_ID', sorting=True).add_partition_key('ORDER_DATE', sorting=True)
    Link_1 (src='Source', dest='Target')

.. _preparing_data__batch__link_partitioning__removing_partition_keys:

Removing Partition Keys
~~~~~~~~~~~~~~~~~~~~~~~~

To remove a partition key from a link, use the :py:meth:`Link.remove_partition_key() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link.remove_partition_key>` method with the column name. This method also returns the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link` object for method chaining.

.. code-block:: python

    >>> link.remove_partition_key('REGION')
    Link_1 (src='Source', dest='Target')

.. _preparing_data__batch__link_partitioning__inspecting_partitioning:

Inspecting Partitioning Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can inspect the partitioning configuration of a link by accessing its ``part_type`` and ``key_cols_part`` attributes.

.. code-block:: python

    >>> link.part_type
    'hash'
    >>> link.perform_sort
    True
    >>> len(link.key_cols_part)
    2
    >>> link.key_cols_part[0]
    {'keyCol': 'CUSTOMER_ID', 'partitioning': True, 'sorting': True, 'ci-cs': 'cs', 'asc-desc': 'asc'}
