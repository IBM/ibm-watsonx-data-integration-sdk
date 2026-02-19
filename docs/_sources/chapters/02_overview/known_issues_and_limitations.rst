.. _overview__known_issues_and_limitations:

Known Issues and Limitations
============================

A list of currently known issues and limitations can be found below.
This page is actively maintained and updated with each release of the ``ibm-watsonx-data-integration`` SDK for Python.

Known Issues
~~~~~~~~~~~~

* Users may encounter HTTP 429 errors ('too many requests') when making calls from the SDK.

* When retrieving a parameter set by name using the :py:meth:`Project.parameter_sets.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method, the ``parameters`` field of the returned :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` object will be empty.

* Changes to batch flow runtime parameters may not show up in the UI. If you are strictly using the SDK this will not be a problem.

* Creating parameter sets with invalid characters (example: ``'MyParameterSet!*'``) can cause parameter set related operations to fail.

* When retrieving subflows or batch flows that contain subflow stages, the subflow DAG will be empty. You can workaround this by downloading the flow and using the :py:class:`~ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator` to create the flow.

* When retrieving subflows that contain subflow stages, you cannot modify that subflow's DAG. You can workaround this by downloading the flow and using the :py:class:`~ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator` to create the flow.

* When using :py:class:`~ibm_watsonx_data_integration.services.datastage.codegen.python_generator.PythonGenerator` on a Batch flow that has runtime settings or parameters changed at the flow level the flow will be correctly generated but the runtime settings and parameters will not.

* Deleting a project's asset using a different project will still delete the object.

* BatchFlows with unsupported stage types can result in the flow not getting serialized into a python object.

* Codegen does not set compile mode for generated BatchFlow.

* Compile mode is not persisted when duplicating BatchFlow.

Limitations
~~~~~~~~~~~
* The SDK only partially supports DataStage (batch processing) functionality at this point. DataStage components will be added in future releases.

* When using the :py:class:`~ibm_watsonx_data_integration.codegen.generator.PythonGenerator` class to generate code, the authenticator object that is generated will always be :py:class:`~ibm_watsonx_data_integration.common.auth.IAMAuthenticator` regardless of what type of authenticator object is passed in to the generator.
