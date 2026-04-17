.. _overview__known_issues_and_limitations:

Known Issues and Limitations
============================

A list of currently known issues and limitations can be found below.
This page is actively maintained and updated with each release of the ``ibm-watsonx-data-integration`` SDK for Python.

Known Issues
~~~~~~~~~~~~

* Users may encounter HTTP 429 errors ('too many requests') when making calls from the SDK.

* Changes to batch flow runtime parameters may not appear in the UI. If you are strictly using the SDK, this will not be a problem.

* When retrieving subflows that contain subflow stages, you cannot modify that subflow's DAG. You can work around this by downloading the flow and using the :py:class:`~ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator` to create the flow.

* When using :py:class:`~ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator` on a batch flow that has runtime settings or parameters changed at the flow level, the flow will be generated correctly, but the runtime settings and parameters will not.

* BatchFlow objects with unsupported stage types can result in the flow not being serialized into a Python object.

* The SDK incorrectly allows connecting outputs to source-only stages in streaming flows. These stages should only act as data sources and cannot accept inputs from other stages, but the SDK does not enforce this restriction.

Limitations
~~~~~~~~~~~
* The SDK only partially supports DataStage (batch processing) functionality at this point. DataStage components will be added in future releases.

* Most SDK models use the ``name`` attribute for display names and searching. However, :py:class:`~ibm_watsonx_data_integration.services.streamsets.models.flow_model.Stages` inconsistently uses ``stage_name`` instead of ``name`` for this purpose.

* Streaming python generator does not support parameter sets.
