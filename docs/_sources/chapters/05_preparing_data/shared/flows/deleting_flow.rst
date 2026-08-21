To delete a flow in the UI, go to **Assets**, choose a flow, click the three dots next to it, and select **Delete**.

.. image:: /_static/images/flows/flow_options.png
   :alt: Screenshot of deleting a flow.
   :align: center
   :width: 100%

To delete a flow using the SDK, pass a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance to the :py:meth:`Project.delete_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_flow>` method.

This method returns an HTTP response indicating the status of the delete operation.

.. skip: start 'not existing duplicated_flow'

.. code-block:: python

    >>> project.delete_flow(duplicated_flow)
    <Response [204]>

.. skip: end
