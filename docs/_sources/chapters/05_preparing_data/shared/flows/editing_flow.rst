You can edit a flow in multiple ways.

For starters, you can edit a flow's attributes like ``name`` or ``description``.

.. skip: start 'not existing new_flow'

.. code-block:: python

    >>> new_flow.description = 'new description for the flow'
    >>> new_flow
    StreamsetsFlow(name='My first flow', description='new description for the flow', ...)

.. skip: end

Also you can edit any flow by editing its stages.
This can include adding a stage, removing a stage, updating a stage's configuration or connecting a stage in a different way than before.
All the operations described are covered in the :ref:`Stages <preparing_data__stages>` documentation.
