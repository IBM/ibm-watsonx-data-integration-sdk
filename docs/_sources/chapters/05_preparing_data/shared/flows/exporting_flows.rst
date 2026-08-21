Exporting Flows
~~~~~~~~~~~~~~~

To export a flow using the SDK, call the :py:meth:`Project.export_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flow>` method
and pass an individual :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` object. If you want to
export multiple flows at the same time, call the :py:meth:`Project.export_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flows>`
method and pass a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` objects.

.. note::

    When using :py:meth:`Project.export_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flows>`
    to export multiple flows at once, all objects in the list must be of the same type. You cannot
    pass a list containing both :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` and
    :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow` objects. It must contain one type or the other.


You can set the following additional parameters:
    * ``with_plain_text_credentials`` – export credentials in plain text. (only relevant to streaming flows)
    * ``destination`` – specify the export location.
    * ``stream`` – stream the ZIP file data.

The function returns the location where the exported ZIP file was written.

.. skip: start 'Export Flows API endpoint not on Prod yet'

.. code-block:: python

    >>> flow
    StreamingFlow(name='My streaming flow', description='optional description', ...)

    >>> project.export_flow(flow=flow)
    PosixPath('flows.zip')

.. skip: end
