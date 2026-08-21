You can edit a flow in multiple ways.

For starters, you can edit a flow's attributes like ``name`` or ``description``.

.. skip: start 'not existing new_flow'

.. code-block:: python

    >>> new_flow.description = 'new description for the flow'
    >>> new_flow
    StreamingFlow(name='My first flow', description='new description for the flow', ...)

.. skip: end

You can also edit any flow by editing its stages.
This can include adding a stage, removing a stage, updating a stage's configuration, or connecting a stage in a different way than before.
All of these operations are covered in the Stages documentation (see :ref:`Batch Stages <preparing_data__batch_stages>` or :ref:`Streaming Stages <preparing_data__streaming_stages>`).
