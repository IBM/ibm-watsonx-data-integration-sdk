Exporting Flows
~~~~~~~~~~~~~~~

To export a flow via the SDK, you need to call the :py:meth:`Project.export_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flow>` method and you
must pass in either an individual :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` object. If you would like to
export multiple flows at the same time, you can call the :py:meth:`Project.export_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flows>`
method pass in a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.flow_model.Flow` objects.

.. note::

    When using :py:meth:`Project.export_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.export_flows>`
    to export multiple flows at once, it is important that all objects in the list must be of the same type. You cannot
    pass in a list of :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.StreamingFlow` and
    `~ibm_watsonx_data_integration.services.datastage.models.flow.BatchFlow` objects. It must be one or the other.


You can set the following additional parameters:
    * ``with_plain_text_credentials`` – export credentials in plain text. (only relevant to streaming flows)
    * ``destination`` – specify the export location.
    * ``stream`` – stream the ZIP file data.

The function will return the location the exported zip file was written to.

.. skip: start 'Export Flows API endpoint not on Prod yet'

.. code-block:: python

    >>> flow
    StreamingFlow(name='My streaming flow', description='optional description', ...)

    >>> project.export_flow(flow=flow)
    PosixPath('flows.zip')

.. skip: end
