.. _preparing_data__batch__flows:

Flows
=====

.. include:: ../shared/flows/introduction.rst

The SDK provides the following functionality to interact with batch flows:
    * Creating a flow
    * Retrieving flows
    * Editing a flow
    * Updating a flow
    * Duplicating a flow
    * Deleting a flow
    * Compiling a flow

.. _preparing_data__batch__flows__creating_a_flow:

Creating a Flow
~~~~~~~~~~~~~~~

In the UI, you can create a flow by navigating to **Assets -> New asset -> Transform and integrate data with DataStage**.

.. image:: /_static/images/flows/create_batch_flow.png
    :alt: Screenshot of the flow creation page.
    :align: center
    :width: 100%

In the SDK, you can create a flow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_flow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_flow>` method.
You are required to supply a ``name`` parameter and an optional ``description`` parameter.
You must also specify a ``flow_type`` as ``'batch'``.
This method returns a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow` instance.

.. code-block:: python

    >>> new_flow = project.create_flow(name='My batch flow', description='optional description', environment=None, flow_type='batch')
    >>> new_flow
    BatchFlow(..., name='My batch flow', description='optional description', flow_id=...)

.. invisible-code-block: python
    project.create_flow(name='My streaming flow', description='optional description', environment=environment)

.. _preparing_data__batch__flows__retrieving_a_flow:

Retrieving Flows
~~~~~~~~~~~~~~~~

Flows can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.flows <ibm_watsonx_data_integration.cpd_models.project_model.Project.flows>` property.
If you wish to only retrieve batch flows use the :py:meth:`Project.flows.get_all() <ibm_watsonx_data_integration.common.models.CollectionModel.get_all>` method to filter by ``flow_type``.
You can also retrieve a single flow using the :py:meth:`Project.flows.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method which requires the ``flow_id`` parameter.


.. code-block:: python

    >>> project.flows  # a list of all the flows
    [StreamingFlow(name='example flow', ...), ...BatchFlow(..., name='example batch flow', ...)...]
    >>> project.flows.get_all(flow_type='batch')
    [...BatchFlow(..., name='example batch flow', ...)...]
    >>> project.flows.get(name='My batch flow')
    BatchFlow(..., name='My batch flow', description='optional description', ...)

.. _preparing_data__batch__flows__editing_a_flow:

Editing a Flow
~~~~~~~~~~~~~~

.. include:: ../shared/flows/editing_flow.rst

.. _preparing_data__batch__flows__updating_a_flow:

Updating a Flow
~~~~~~~~~~~~~~~

.. include:: ../shared/flows/updating_flow.rst

.. _preparing_data__batch__flows__duplicating_a_flow:

Duplicating a Flow
~~~~~~~~~~~~~~~~~~

.. include:: ../shared/flows/duplicating_flow.rst

.. _preparing_data__batch__flows__deleting_a_flow:

Deleting a Flow
~~~~~~~~~~~~~~~

.. include:: ../shared/flows/deleting_flow.rst

.. _preparing_data__batch__flows__compiling_a_flow:

Compiling a Flow
~~~~~~~~~~~~~~~~

In the UI, you can compile a batch flow by hitting the 'Compile' icon to compile the flow.
Compiling is required before running a flow, so pressing the 'Run' icon in the UI will automatically compile first.

.. image:: /_static/images/flows/compile_batch_flow.png
   :alt: Screenshot of the compile button.
   :align: center
   :width: 100%

Because of this UI behavior, in the SDK, making a job out of a Batch flow will **automatically compile** the flow for you.
However, if you wish to just compile a flow without creating or running a job for it, you can still call the :py:meth:`BatchFlow.compile() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.compile>` method.
This will return an HTTP response indicating the status of the compile operation.

.. _preparing_data__batch__flows__setting_runtime_parameters_of_a_flow:

Setting Runtime Parameters of a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can set the runtime parameters of a flow by clicking the settings button and going to the runtime parameters tab. This tab will only show up if the flow has a local parameter or is using a parameter set.
In this tab you can choose what value set you want to use for your parameter sets or change the values of specific parameters.

.. image:: /_static/images/flows/open_flow_settings.png
   :alt: Screenshot of the settings button.
   :align: center
   :width: 100%

.. image:: /_static/images/flows/runtime_parameters.png
   :alt: Screenshot of the runtime parameters tab.
   :align: center
   :width: 100%

In the SDK, there are a few different methods for changing runtime parameters.

:py:meth:`BatchFlow.set_runtime_value_set() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.set_runtime_value_set>` takes in the ``parameter_set_name`` and the ``value_set_name`` you want to use for that parameter set.

:py:meth:`BatchFlow.set_runtime_parameter_value() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.set_runtime_parameter_value>` takes in the ``parameter_set_name``, the ``parameter_name``, and the ``value`` you wish to set that parameter to.

:py:meth:`BatchFlow.set_runtime_local_parameter() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.set_runtime_local_parameter>` takes in the ``local_parameter_name`` and the ``value`` you wish to set it to.

.. skip: start 'ignore return'

.. code-block:: python

    >>> # batch_flow_with_parameters is a batch flow that uses a parameter set named myparamset and a local variable named localparam1
    >>> batch_flow_with_parameters.set_runtime_value_set(parameter_set_name='myparamset', value_set_name='value_set_1')
    >>> batch_flow_with_parameters.set_runtime_parameter_value(parameter_set_name='myparamset', parameter_name='param1', value='default')
    >>> batch_flow_with_parameters.set_runtime_local_parameter(local_parameter_name='localparam1', value='random')

.. skip: end

.. warning::
   Changes to runtime parameters may not show up in the UI. If you are strictly using the SDK this will not be a problem.

.. _preparing_data__batch__flows__setting_runtime_settings_of_a_flow:

Setting Runtime Settings of a Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the same settings window mentioned in the previous section, there is another tab called Run. In this tab there are runtime settings that you can change for a flow. There is also an NLS and Format tab but these are not implemented in the SDK yet.
These settings include the environment for the flow, the warning limit, and the max job retention. These settings (along with the runtime parameters) can also be changed per job, which is explained in this section: :ref:`Editing Runtime Settings <projects__jobs__editing_a_batch_job>`.

.. image:: /_static/images/flows/runtime_settings.png
   :alt: Screenshot of the runtime settings tab.
   :align: center
   :width: 100%


In the SDK, you can change the runtime settings of a flow by directly editing the ``configuration`` property of a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow`. There are 4 different fields you can change.

* ``environment``: The internal name of the batch environment to use. To find the internal name of a batch environment you can either list all internal names with :py:meth:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.list_batch_environments` or call :py:meth:`~ibm_watsonx_data_integration.cpd_models.project_model.Project.get_batch_environment` with the display name.
* ``warn_limit``: The number of warnings before the stages are stopped. Takes an ``int`` greater than 0 or None for no limit.
* ``retention_days``: The number of days to keep a job run. Cannot be set if ``retention_amount`` is also set. Takes an ``int`` greater than 0 or None for no limit.
* ``retention_amount``: The number of job runs to keep in total. Cannot be set if ``retention_days`` is also set. Takes an ``int`` greater than 0 or None for no limit.

.. code-block:: python

    >>> batch_flow.configuration.environment='default_datastage_px'
    >>> batch_flow.configuration.warn_limit=20
    >>> batch_flow.configuration.retention_days=15

.. warning::
   Changes to runtime settings may not show up in the UI. If you are strictly using the SDK this will not be a problem.


.. _preparing_data__batch__flows__exporting_a_flow:

.. include:: ../shared/flows/exporting_flows.rst

.. _preparing_data__batch__flows__importing_flows:

Importing Flows
~~~~~~~~~~~~~~~

To import a flow via the SDK using a zip file, you need to call the :py:meth:`Project.import_flows() <ibm_watsonx_data_integration.cpd_models.project_model.Project.import_flows>` method and you
must specify the ``source`` parameter, which is the Path to the zip file containing the json(s) of the batch flow(s) to be imported and the ``type`` parameter, which should be
``'batch'``.

You can also specify the following parameters to configure the import to better fit your needs:

    * ``'on_failure'`` – Option for error handling during import. See :py:class:`~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportOnFailure` for available options.
    * ``'conflict_resolution'`` – Option for handling conflict resolutions during import. See :py:class:`~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportConflictResolution` for available options.
    * ``'import_only'`` – If true, will only import the batch flows and will not compile them.
    * ``'include_dependencies'`` – If true, will not also import dependencies that the flow(s) have in the import file.
    * ``'replace_mode'`` – Option for replace mode method when conflict resolution is set to ``'replace'``. See :py:class:`~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportReplaceMode` for available options.
    * ``'import_binaries'`` – If set to true, will also import compiled binaries found in the zip file.
    * ``'wait'`` – Specified wait period. See below for more details.


The function will return a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportResponse`
object which will contain information about the import request. It is important to note that importing batch flows is an
asynchronous action, meaning that once you have received the response object the import might not be finished. The ``wait``
parameter allows you to manage this in different ways depending on its value:

    * ``-1``: The default option, which will wait until the import is complete.
    * ``0>``: Time, in seconds, to wait for the import to finish.
    * ``None`` or ``0``: No wait time, will return immediately.


.. skip: start 'Do not actually import a flow'

.. code-block:: python

    >>> # wait until import has completed
    >>> resonse = project.import_flows(type='batch', source='flows_to_import.zip', wait=-1)
    >>> response.status
    BatchImportStatus.COMPLETED

    >>> # no wait time
    >>> resonse = project.import_flows(type='batch', source='flows_to_import.zip', wait=None)
    >>> response.status
    BatchImportStatus.STARTED

    >>> # wait for 10 seconds
    >>> resonse = project.import_flows(type='batch', source='flows_to_import.zip', wait=10)
    >>> response.status
    BatchImportStatus.IN_PROGRESS

.. skip: end

The import's status can be confirmed by the :py:attr:`BatchImportResponse.status <~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportResponse.status>`
field. If the import is not completed and you would like to get the latest status, you can use the
:py:meth:`BatchImportResponse.refresh() <~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportResponse.refresh>`
method.

.. skip: start 'Not performing actual import of flows'

.. code-block:: python

    >>> # no wait time
    >>> resonse = project.import_flows(type='batch', source='flows_to_import.zip', wait=None)
    >>> response.status
    BatchImportStatus.STARTED

    >> time.sleep(100) # with the assumption the import will be complete
    >> response = response.refresh()
    >> response.status
    BatchImportStatus.COMPLETED

.. skip: end

Lastly, you can retrieve the list of imported flows from the
:py:meth:`BatchImportResponse.imported_flows() <~ibm_watsonx_data_integration.services.datastage.models.import_response_model.BatchImportResponse.imported_flows>`
method, which will return an iterator of :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow`
objects. You can specify the optional parameter ``load=True`` to retrieve the DAGs for each of the flows.
Otherwise, the :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow` objects
that are returned will only have the ``flow_id`` and ``name`` fields populated.

.. skip: start 'Not performing actual import of flows'

.. code-block:: python

    >>> # no wait time
    >>> resonse = project.import_flows(type='batch', source='flows_to_import.zip', wait=-1)
    >>> response.status
    BatchImportStatus.STARTED

    >>> response.imported_flows()
    <list_iterator object at 0x107385960>

.. skip: end
