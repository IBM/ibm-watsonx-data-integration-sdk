In the UI, you can update a flow by making changes to the flow and hitting the 'Save' icon to update the flow.

.. image:: /_static/images/flows/save_flow_button.png
   :alt: Screenshot of updating the flow.
   :align: center
   :width: 100%

In the SDK, you can make any changes to a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance
in memory and update it by passing this object to :py:meth:`Project.update_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_flow>` method.

This method returns an HTTP response indicating the status of the update operation.

.. skip: start 'not existing new_flow'

.. code-block:: python

    >>> new_flow.name = 'new flow name'  # you can also update the stages, configuration, etc.
    >>> project.update_flow(new_flow)
    <Response [200]>
    >>> new_flow
    StreamingFlow(name='new flow name', description='new description for the flow', ...)

.. skip: end
