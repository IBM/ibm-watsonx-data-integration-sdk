.. _preparing_data__parameter_sets:

Parameter Sets
==============

A parameter set is an object that helps make your jobs more flexible and reusable. Parameter sets allow you to specify values at run time rather than hardcoding them.

The SDK provides functionality to interact with parameter sets.

This includes operations such as:
    * Creating a parameter set
    * Retrieving a parameter set
    * Updating a parameter set
    * Adding value sets
    * Adding local parameters
    * Using local parameters and parameter sets
    * Duplicating a parameter set
    * Deleting a parameter set

.. _preparing_data__parameter_sets__creating_a_parameter_set:

Creating a Parameter Set
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can create a new Parameter Set by navigating to **Assets -> New asset -> Define reusable sets of parameters**.

.. image:: /_static/images/connections/create_connection.png
   :alt: Screenshot of the Parameter set creation in the UI - Step 1
   :align: center
   :width: 100%

.. image:: /_static/images/parameter_sets/create_paramset.png
   :alt: Screenshot of the Parameter set creation in the UI - Step 2
   :align: center
   :width: 100%


In the SDK, you can create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` object within a
:py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project`,
by selecting the appropriate project from the :py:class:`~ibm_watsonx_data_integration.platform.Platform` and then using
the :py:meth:`Project.create_parameter_set() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_parameter_set>` method to instantiate the Parameter Set.

You must provide a ``name`` to create a parameter set. The parameter set name can only contain letters, numbers, and underscores.
When adding parameters to a parameter set, you should use the :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterType` enum to specify the parameter type.
The available parameter types match options available in UI.

.. invisible-code-block: python

    >>> from ibm_watsonx_data_integration.cpd_models.parameter_set_model import ParameterType
    >>> params = project.parameter_sets
    >>> for i in params:
    ...     if i.name == 'testparamset':
    ...         project.delete_parameter_set(i)

.. code-block:: python

    >>> paramset = project.create_parameter_set('testparamset')
    >>> paramset.add_parameter(parameter_type=ParameterType.Integer, name='qty', value='100')
    ParameterSet(name='testparamset', parameters=[Parameter(name='qty', param_type='int64', value='100')], description='', value_sets=[])
    >>>
    >>> project.update_parameter_set(paramset)
    <Response [200]>

When creating a parameter set with :py:meth:`Project.create_parameter_set() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_parameter_set>`, you can provide parameters as either :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.Parameter` objects or as dictionaries.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.cpd_models.parameter_set_model import Parameter, ParameterType
    >>>
    >>> # Using Parameter objects
    >>> param1 = Parameter(name='qty', param_type=ParameterType.Integer, value='100')
    >>> param2 = Parameter(name='score', param_type=ParameterType.String, value='high')
    >>> paramset_a = project.create_parameter_set(name='my_params_from_objects', parameters=[param1, param2])
    >>>
    >>> # Using dictionaries
    >>> params_dict = [
    ...     {'name': 'qty', 'type': ParameterType.Integer, 'value': '100'},
    ...     {'name': 'score', 'type': ParameterType.String, 'value': 'high'}
    ... ]
    >>> paramset_b = project.create_parameter_set(name='my_params_from_dictionaries', parameters=params_dict)

.. _preparing_data__parameter_sets__retrieving_an_existing_parameter_set:

Retrieving an Existing Parameter Set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can get all Parameter Sets by navigating to **Assets -> Configurations -> Parameter sets**.

.. image:: /_static/images/parameter_sets/get_paramsets.png
   :alt: Screenshot of the Connection listing in the UI
   :align: center
   :width: 100%

In the SDK, Parameter Sets can be retrieved using the :py:class:`Project.parameter_sets <ibm_watsonx_data_integration.cpd_models.project_model.Project.parameter_sets>` property.
This property returns a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` objects.

.. code-block:: python

    >>> # Returns a list of all parameter sets
    >>> project.parameter_sets
    [...ParameterSet(name='testparamset', parameters=[Parameter(name='qty', param_type='int64', value='100')], description='', value_sets=[])...]
    >>>
    >>> project.parameter_sets.get(name='testparamset')
    ParameterSet(name='testparamset', parameters=[Parameter(name='qty', param_type='int64', value='100')], description='', value_sets=[])


.. _preparing_data__parameter_sets__updating_a_parameter_set:

Updating a Parameter Set
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can update a Parameter Set by navigating to **Assets -> Configurations -> Parameter sets** and clicking the name of the Parameter Set you want to edit.
This will take you to a new page where you can add, edit, and delete parameters.

.. image:: /_static/images/parameter_sets/update_paramset1.png
   :alt: Screenshot of the Parameter set updating in the UI - Step 1
   :align: center
   :width: 100%

.. image:: /_static/images/parameter_sets/update_paramset2.png
   :alt: Screenshot of the Parameter set updating in the UI - Step 2
   :align: center
   :width: 100%

To add parameters to a Parameter Set in the SDK, you can use the :py:meth:`ParameterSet.add_parameter() <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet.add_parameter>` method.
This method requires a ``name`` and ``parameter_type`` and can also take a ``value``, ``description``, ``prompt``, and ``valid_values``. The ``valid_values`` argument is only used in a parameter of type ``list``.

To remove parameters use the :py:meth:`ParameterSet.remove_parameter() <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet.remove_parameter>` method which takes in the ``name`` of the parameter you wish to remove.

.. code-block:: python

    >>> paramset = (
    ...    paramset.add_parameter(parameter_type=ParameterType.String, name='score', value='person')
    ...    .add_parameter(parameter_type=ParameterType.Date, name='date', value='2025-03-12')
    ... )
    >>> paramset.remove_parameter('score')
    ParameterSet(name='testparamset', parameters=[Parameter(name='qty', param_type='int64', value='100'), Parameter(name='date', param_type='date', value='2025-03-12')], description='', value_sets=[])


You can also directly change the ``name`` and ``description`` of the Parameter Set. When changing the parameter set name, remember that it can only contain letters, numbers, and underscores.
To apply these modifications to the Parameter Set we pass the instance to the :py:meth:`Project.update_parameter_set() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_parameter_set>` method.

This method returns an HTTP response indicating the status of the update operation.

.. code-block:: python

    >>> paramset.name = 'New_Paramset_Name'
    >>> project.update_parameter_set(paramset)
    <Response [200]>
    >>>
    >>> project.parameter_sets.get(name='New_Paramset_Name')
    ParameterSet(name='New_Paramset_Name', parameters=[...Parameter(...)...], description='', value_sets=[])

Updating Parameters Within a Parameter Set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you add a parameter with an existing name to a :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet`, it will overwrite the existing parameter. Alternatively, you can fetch a single parameter and modify it directly:

.. code-block:: python

    >>> # Overwriting an existing parameter by adding with the same name
    >>> paramset.add_parameter(parameter_type=ParameterType.Integer, name='qty', value='200')
    ParameterSet(name='New_Paramset_Name', parameters=[Parameter(name='qty', param_type='int64', value='200'), ...], description='', value_sets=[])
    >>>
    >>> # Or fetch and modify a parameter directly
    >>> param = next((p for p in paramset.parameters if p.name == 'qty'), None)
    >>> if param:
    ...     param.value = '300'
    >>> project.update_parameter_set(paramset)
    <Response [200]>

.. _preparing_data__parameter_sets__adding_value_sets:

Adding Value Sets
~~~~~~~~~~~~~~~~~

Value sets let you assign different values to the parameters in a parameter set. In the UI, to create a value set first navigate to **Assets -> Configurations -> Parameter sets** and click the name of the Parameter Set you want to edit. Then, click the Value sets tab to create, edit, or delete a value set.

.. image:: /_static/images/parameter_sets/value_set1.png
   :alt: Screenshot of the value set tab in the UI - Step 1
   :align: center
   :width: 100%

Here is the window that pops up when you click create value set.

.. image:: /_static/images/parameter_sets/value_set2.png
   :alt: Screenshot of the value set window in the UI - Step 2
   :align: center
   :width: 100%

In the SDK, you can create a :py:class:`ValueSet <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ValueSet>` which requires the ``name`` of the value set. Then, add values to the value set by calling :py:meth:`ValueSet.add_value() <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ValueSet.add_value>` which takes in the ``name`` of the parameter and the ``value`` for the parameter. Finally, add the value set to the parameter set by passing the ValueSet object into :py:meth:`ParameterSet.add_value_set() <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet.add_value_set>`.

If a parameter is not given a value its default value will be used. You can also call :py:meth:`ValueSet.remove_value() <ibm_watsonx_data_integration.cpd_models.parameter_set_model.ValueSet.remove_value>` with the ``name`` of the parameter to reset it to the default value.

.. code-block:: python

    >>> from ibm_watsonx_data_integration.cpd_models.parameter_set_model import ValueSet
    >>> set_1 = (
    ...    ValueSet(name='set_1')
    ...    .add_value(name='qty', value='20')
    ...    .add_value(name='date', value='2022-07-13')
    ... )
    >>> set_1.remove_value(name='date')
    ValueSet(name='set_1', values=[{'name': 'qty', 'value': '20'}])
    >>>
    >>> paramset.add_value_set(set_1)
    ParameterSet(name='New_Paramset_Name', parameters=[Parameter(name='qty', param_type='int64', value='300'), Parameter(name='date', param_type='date', value='2025-03-12')], description='', value_sets=[ValueSet(name='set_1', values=[{'name': 'qty', 'value': '20'}, {'name': 'date', 'value': '2025-03-12'}])])
    >>>
    >>> set_2 = (
    ...    ValueSet(name='set_2')
    ...    .add_value(name='qty', value='40')
    ... )
    >>> paramset.add_value_set(set_2)
    ParameterSet(name='New_Paramset_Name', parameters=[Parameter(name='qty', param_type='int64', value='300'), Parameter(name='date', param_type='date', value='2025-03-12')], description='', value_sets=[ValueSet(name='set_1', values=[{'name': 'qty', 'value': '20'}, {'name': 'date', 'value': '2025-03-12'}]), ValueSet(name='set_2', values=[{'name': 'qty', 'value': '40'}, {'name': 'date', 'value': '2025-03-12'}])])

In this example both value sets have the ``qty`` parameter set to a new value while the ``date`` parameter is still the default value.

You can choose what value set you want a job to use by :ref:`Editing the runtime settings <projects__jobs__editing_a_batch_job>` of the job.

.. note::
    In the UI, there is a way to choose a value set for a flow but this is not yet implemented in the SDK.


.. _preparing_data__parameter_sets__adding_local_parameters:

Adding local Parameters
~~~~~~~~~~~~~~~~~~~~~~~

Local parameters are parameters that only exist within a single flow.

In the UI, you can add a local parameter by navigating to **Flow parameters -> Local parameters -> Create parameter**

.. image:: /_static/images/parameter_sets/add_parameters1.png
   :alt: Screenshot of adding local parameters in the UI - Part 1
   :align: center
   :width: 100%

.. image:: /_static/images/parameter_sets/add_parameters2.png
   :alt: Screenshot of adding local parameters in the UI - Part 2
   :align: center
   :width: 100%

In the SDK, you can call the :py:meth:`BatchFlow.add_local_parameter() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.add_local_parameter>` method.
Just like the add_parameter method above, this method requires a ``name`` and ``parameter_type`` and can also take a ``value``, ``description``, ``prompt``, and ``valid_values``.

.. code-block:: python

    >>> batch_flow.add_local_parameter(parameter_type=ParameterType.String, name='tablename', value='cust_source') # doctest: +SKIP
    >>> batch_flow.add_local_parameter(parameter_type=ParameterType.List, name='shoplist', valid_values=['tomato', 'lettuce', 'bacon'], value='bacon') # doctest: +SKIP

.. _preparing_data__parameter_sets__using_local_parameters_and_parameter_sets:

Using local parameters and Parameter Sets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can use a Parameter Set in a flow by navigating to **Flow parameters -> Parameter sets -> Add parameter sets** and choosing the desired parameter set from the list.

.. image:: /_static/images/parameter_sets/use_paramset.png
   :alt: Screenshot of the Connection deletion in the UI
   :align: center
   :width: 100%

In the SDK to use a Parameter Set instance in a flow you can pass it to :py:meth:`BatchFlow.use_parameter_set() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.use_parameter_set>` method.
For local parameters there are no additional actions that you need to take after you :ref:`Add a local parameter <preparing_data__parameter_sets__adding_local_parameters>`.

.. code-block:: python

    >>> batch_flow.use_parameter_set(paramset) # doctest: +SKIP


To use a parameter from a parameter set in your code you can use the following notation: ``'#New Paramset Name.schema#'`` where the name of the parameter set is followed by the parameter you wish to use.
For local parameters you do not need to put anything in front of the parameter name: ``'#tablename#'``.

.. code-block:: python

    >>> bigquery = batch_flow.add_stage('Google BigQuery', 'BigQuery')
    >>> bigquery.configuration.table_name = '#tablename#'
    >>> bigquery.configuration.dataset_name = '#testparamset.schema#'


.. _preparing_data__parameter_sets__duplicating_a_parameter_set:

Duplicating a Parameter Set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can duplicate a Parameter Set by navigating to **Assets -> Configurations -> Parameter sets**.
Then click the three-dot menu on the Parameter Set you want to duplicate and select **Duplicate**.
A new Parameter Set with a ``_copy_1`` suffix appended to the name will be created.

.. image:: /_static/images/parameter_sets/duplicate_paramset.png
   :alt: Duplicate Parameter Set
   :align: center
   :width: 100%

In the SDK, you can duplicate a Parameter Set by passing it to the :py:meth:`Project.duplicate_parameter_set() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_parameter_set>` method.
This method returns a new :py:class:`~ibm_watsonx_data_integration.cpd_models.parameter_set_model.ParameterSet` object with all parameters and value sets copied from the original.

.. code-block:: python

    >>> original_paramset = project.parameter_sets.get(name='New_Paramset_Name')
    >>> duplicated_paramset = project.duplicate_parameter_set(original_paramset)
    >>> duplicated_paramset.name
    'New_Paramset_Name_copy_1'

.. _preparing_data__parameter_sets__deleting_a_parameter_set:

Using PROJDEF
~~~~~~~~~~~~~
You can use the PROJDEF parameter set to contain the parameters and environment variable values for a Batch flow, job, or job run to reference.
You can read more about PROJDEF `here <https://dataplatform.cloud.ibm.com/docs/content/dstage/dsnav/topics/projdef-parameter-set.html?context=cpdaas>`_.
To add PROJDEF in the UI go to **Flow parameters -> Add PROJDEF parameter**

.. image:: /_static/images/parameter_sets/use_projdef.png
   :alt: Screenshot of using PROJDEF in UI
   :align: center
   :width: 100%

To use PROJDEF in the SDK you first need to create parameter_set with name 'PROJDEF' and then use it like a normal parameterSet.

.. code-block:: python

    >>> projdef_param = project.create_parameter_set('PROJDEF')
    >>> projdef_param.add_parameter(parameter_type=ParameterType.Integer, name='qty', value='100')
    ParameterSet(name='PROJDEF', parameters=[Parameter(name='qty', param_type='int64', value='100')], description='', value_sets=[])
    >>>
    >>> project.update_parameter_set(projdef_param)
    <Response [200]>
    >>>
    >>> batch_flow.use_parameter_set(projdef_param)
    BatchFlow(...)
    >>>
    >>> project.update_flow(batch_flow)
    <Response [201]>


To add only some of the PROJDEF parameters use `flow.use_projdef_parameter()`.

.. code-block:: python

    >>> projdef_param = project.parameter_sets.get(name='PROJDEF')
    >>> projdef_param = project.parameter_sets.get(parameter_set_id=projdef_param.parameter_set_id)
    >>>
    >>> param = next((p for p in projdef_param.parameters if p.name == 'qty'), None)
    >>> batch_flow.use_projdef_parameter(param)
    BatchFlow(...)
    >>>
    >>> project.update_flow(batch_flow)
    <Response [201]>


Deleting a Parameter Set
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can delete an existing Parameter Set by navigating to the **Assets -> Configurations -> Parameter sets**.

.. image:: /_static/images/parameter_sets/delete_paramset.png
   :alt: Screenshot of the Parameter Set deletion in the UI
   :align: center
   :width: 100%

In the SDK to delete a Parameter Set instance you can pass it to :py:meth:`Project.delete_parameter_set() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_parameter_set>` method.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> project.delete_parameter_set(paramset)
    <Response [204]>
