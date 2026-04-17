To duplicate a flow using the SDK, pass a :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` instance
to the :py:meth:`Project.duplicate_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_flow>` method,
along with the ``name`` parameter for the new flow and an optional ``description`` parameter.

This duplicates the flow and returns a new instance of :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow`.

.. skip: start 'not existing new_flow'

.. code-block:: python

    >>> duplicated_flow = project.duplicate_flow(new_flow, name='duplicated flow', description='duplicated flow description')
    >>> duplicated_flow
    StreamsetsFlow(name='duplicated flow', description='duplicated flow description', ...)

.. skip: end
