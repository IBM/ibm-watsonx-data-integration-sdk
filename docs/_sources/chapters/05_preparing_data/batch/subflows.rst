.. _preparing_data__subflows:

Subflows
========


A subflow is a batch flow component that can be used to simplify a complex design visually to make it easier to
understand and/or act as a reusable component by other data flows.

There are two primary types of subflows:
    * Local Subflow: A subflow that is local to the data flow you are working on. It is not its own asset and cannot be used by other flows.
    * (Global) Subflow: A subflow that that can be reused by other data flows in the project. It is not local to any specific data flow.

.. note::
    In the documentation, global subflow assets are referred to as 'subflows', while local subflows will always be explicitly referred to as 'local subflows'.

The SDK provides the following functionality to interact with subflows:
    * Creating a subflow
    * Retrieving a subflow
    * Editing a subflow
    * Duplicating a subflow
    * Deconstructing a subflow
    * Deleting a subflow
    * Using subflows in flows

.. _preparing_data__subflows__prerequisites:

Prerequisites
~~~~~~~~~~~~~

To create a subflow using the SDK, please make sure you have done the following steps:
    * :ref:`Created a project<projects__projects__creating_a_project>`.

To create a local subflow using the SDK, please make sure you have done the following steps:
    * :ref:`Created a project<projects__projects__creating_a_project>`.
    * :ref:`Created a flow<preparing_data__batch__flows__creating_a_flow>`.

.. note::
    Subflows are a BatchFlow only feature.

.. _preparing_data__subflows__creating_a_subflow:

Creating a Subflow
~~~~~~~~~~~~~~~~~~

.. _preparing_data__subflows__creating_a_subflow__creating_a_subflow:

Creating a Subflow
^^^^^^^^^^^^^^^^^^
In the UI, you can create a flow by navigating to **Assets -> New asset -> Create reusable DataStage components -> Subflow**.

Here, you will be required to choose a name for your subflow along with a name and description.

.. image:: /_static/images/subflows/create_subflow.png
   :alt: Screenshot of the subflow creation page.
   :align: center
   :width: 100%


In the SDK, you can create a subflow from a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:meth:`Project.create_subflow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.create_subflow>` method.
You are required to supply a ``name`` and an optional ``description``.
This method will return a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> new_subflow = project.create_subflow(name='My first subflow', description='optional description')
    >>> new_subflow
    Subflow(name='My first flow', description='optional description')

.. skip: end

.. _preparing_data__subflows__creating_a_subflow__creating_a_local_subflow:

Creating a Local Subflow
^^^^^^^^^^^^^^^^^^^^^^^^

In the UI, you can create a local subflow by highlighting stages in a batch flow and selecting **Create local subflow**.
This creates a local subflow with the nodes you have selected.

.. image:: /_static/images/subflows/create_local_subflow.png
   :alt: Screenshot of creating a local subflow option.
   :align: center
   :width: 100%


In the SDK, you can create a local subflow from a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow` object using the
:py:meth:`BatchFlow.create_subflow() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.create_subflow>` method.
You are required to supply a ``label`` and an optional set of ``nodes`` to create the subflow from.
This method will return a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> myflow # batch flow already created
    BatchFlow(name='My flow', description='')
    >>> new_subflow = myflow.create_subflow(label='subflow_stage')

.. skip: end

.. _preparing_data__subflows__retrieving_a_subflow:

Retrieving Subflows
~~~~~~~~~~~~~~~~~~~

Subflows can be retrieved through a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the
:py:attr:`Project.subflows <ibm_watsonx_data_integration.cpd_models.project_model.Project.subflows>` property.
You can also retrieve a single subflow using the :py:meth:`Project.subflows.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>` method
which takes key word arguments and finds the first subflow match.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> project.subflows # a list of all the flows
    [...Subflow(name='My first subflow', description='optional description')...]

    >>> project.subflows.get(name='My first subflow') # get a subflow with the name 'My first subflow'
    Subflow(name='My first subflow', description='optional description')

.. skip: end

.. note::
    Local subflows cannot be retrieved using this method since they are local to an individual flow rather than a project.

.. _preparing_data__subflows__editing_a_subflow:

Editing a Subflow
~~~~~~~~~~~~~~~~~

.. _preparing_data__subflows__editing_a_subflow__editing_attributes:

Editing Attributes
^^^^^^^^^^^^^^^^^^
You can edit a subflow's attributes like ``name`` or ``description``.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> new_subflow.description = 'new description for the subflow'
    >>> new_subflow
    Subflow(name='My first subflow', description='new description for the subflow')

.. skip: end

Finally, you can edit a flow by editing its stages.
This can include adding a stage, removing a stage, updating a stage's configuration or connecting a stage in a different way than before.
Most of the operations described are covered in :ref:`Stage <preparing_data__stages>` documentation.

.. _preparing_data__subflows__editing_a_subflow__unique_stages_to_subflows:

Unique Stages to Subflows
^^^^^^^^^^^^^^^^^^^^^^^^^

There are two stages that are unique to subflows:
    * :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.EntryNode`: acts as an entry point into the subflow
    * :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.dag.ExitNode`: acts as an exit point into the subflow

.. image:: /_static/images/subflows/entry_exit_nodes.png
   :alt: Screenshot of the entry and exit node stage options.
   :align: center
   :width: 50%


Subflows cannot be utilized within a parent flow without entry and exit nodes. A subflow has no limit to the amount of entry or exit nodes
it can have. To add an entry or exit node to a subflow, you can use the
:py:meth:`Subflow.add_entry_node() <ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow.add_entry_node>`
or :py:meth:`Subflow.add_exit_node() <ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow.add_exit_node>`
method respectively.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> new_subflow
    Subflow(name='My first subflow', description='optional description', ...)
    >>> entry_node = new_subflow.add_entry_node()
    >>> exit_node = new_subflow.add_exit_node()

.. skip: end

.. _preparing_data__subflows__updating_a_subflow:

Updating a Subflow
~~~~~~~~~~~~~~~~~~

.. _preparing_data__subflows__updating_a_subflow__updating_a_subflow:

Updating a Subflow
^^^^^^^^^^^^^^^^^^
In the UI, you can update a flow by making changes to the subflow and hitting the 'Save' icon to update the flow.

.. image:: /_static/images/subflows/save_subflow.png
   :alt: Saving a subflow.
   :align: center
   :width: 100%


In the SDK, you can make any changes to a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance
in memory and update it by passing this object to :py:meth:`Project.update_subflow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_subflow>` method.

.. _preparing_data__subflows__updating_a_subflow__updating_a_local_subflow:

Updating a Local Subflow
^^^^^^^^^^^^^^^^^^^^^^^^

In the UI, you can update a local subflow by making changes to the subflow and saving the parent flow. For more information
on this, please refer to :ref:`Updating a Flow<preparing_data__batch__flows__updating_a_flow>`.

.. _preparing_data__subflows__duplicating_a_subflow:

Duplicating a Subflow
~~~~~~~~~~~~~~~~~~~~~

.. _preparing_data__subflows__duplicating_a_subflow__duplicating_a_subflow:

Duplicating a Subflow
^^^^^^^^^^^^^^^^^^^^^

To duplicate a subflow using the SDK, you need to pass a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance
to the :py:meth:`Project.duplicate_subflow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.duplicate_subflow>` method
along with the ``name`` parameter for the name of the new flow and an optional ``description`` parameter.

This will duplicate a flow and return a new instance of :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow`.

.. skip: start 'dummy subflow'

.. code-block:: python

    >> duplicated_subflow = project.duplicate_subflow(new_subflow, name='duplicated_subflow', description='optional_description')
    >> duplicated_subflow
    Subflow(name='duplicated_subflow', description='optional_description', ...)

.. skip: end

.. _preparing_data__subflows__duplicating_a_subflow__duplicating_a_local_subflow:

Duplicating a Local Subflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^

To duplicate a local subflow, you need to pass a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance
to the  :py:meth:`BatchFlow.duplicate_subflow() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.duplicate_subflow>` method
along with the ``name`` parameter.

.. skip: start 'dummy subflow'

.. code-block:: python

    >> duplicated_subflow = flow.duplicate_subflow(new_subflow, name='duplicated_subflow')
    >> duplicated_subflow
    Subflow(name='duplicated_subflow', description='', ...)

.. skip: end

.. _preparing_data__subflows__deleting_a_subflow:

Deleting a Subflow
~~~~~~~~~~~~~~~~~~

.. _preparing_data__subflows__deleting_a_subflow__deleting_a_subflow:

Deleting a Subflow
^^^^^^^^^^^^^^^^^^

To delete a subflow in the UI, you can go to **Assets**, choose a flow and click on the three dots next to it and choose ``Delete``.

.. image:: /_static/images/subflows/delete_subflow.png
   :alt: Screenshot of deleting subflow.
   :align: center
   :width: 100%


To delete a flow via the SDK, you need to pass a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance
to the :py:meth:`Project.delete_subflow() <ibm_watsonx_data_integration.cpd_models.project_model.Project.delete_subflow>` method.

This method returns an HTTP response indicating the status of the update operation.

.. skip: start 'dummy subflow'

.. code-block:: python

    >>> project.delete_subflow(duplicated_subflow)
    <Response [204]>

.. skip: end

.. _preparing_data__subflows__deleting_a_subflow__deleting_a_local_subflow:

Deleting a Local Subflow
^^^^^^^^^^^^^^^^^^^^^^^^

To delete a local subflow from a flow in the SDK, you need to pass a :py:class:`~ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow` instance
to the :py:meth:`BatchFlow.delete_subflow() <ibm_watsonx_data_integration.services.datastage.models.flow.batch_flow.BatchFlow.delete_subflow>` method.

This method returns None if there were no issues.


.. _preparing_data__subflows__using_subflows_in_flows:

Using Subflows in Flows
~~~~~~~~~~~~~~~~~~~~~~~

To use a subflow in a parent data flow, it must be properly linked, which involves mapping its inputs and outputs.
Consider the following subflow:

.. image:: /_static/images/subflows/subflow_example.png
   :alt: Example subflow.
   :align: center
   :width: 50%


Pay special attention to the name of the links, because when we use this subflow in a parent flow, we need to indicate
to the parent node what links to map to. The following data flow uses this subflow:

.. image:: /_static/images/subflows/example_usage.png
   :alt: Flow using a subflow.
   :align: center
   :width: 50%


To indicate that the input row generator is connected to the entry node **Link_2**, you must indicate that in the input
mapping.

.. image:: /_static/images/subflows/usage_inputs.png
   :alt: Flow using a subflow.
   :align: center
   :width: 50%


You must also do this for the outputs.

.. image:: /_static/images/subflows/usage_outputs.png
   :alt: Flow using a subflow.
   :align: center
   :width: 50%


In the SDK, to link a stage from the parent flow to a subflow's entry node, you must use the
:py:meth:`Link.map_to_link() <ibm_watsonx_data_integration.services.datastage.models.flow.dag.Link.map_to_link>` method,
specifying which link the parent flow should map to.

The following code shows how to create the above subflow and flow:

.. skip: start 'dummy subflow'

.. code-block:: python

    from ibm_watsonx_data_integration.services.datastage import *
    from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
    from ibm_watsonx_data_integration import *

    api_key = '<TODO: insert your api_key>' # pragma: allowlist secret
    auth = IAMAuthenticator(api_key=api_key, base_auth_url='https://cloud.ibm.com') # pragma: allowlist secret
    platform = Platform(auth, base_api_url='https://api.ca-tor.dai.cloud.ibm.com')
    project = platform.projects.get(project_id='<TODO: insert your project_id>')


    flow = project.create_flow(name='subflow_middle_codegen', environment=None, flow_type='batch')

    # Stages
    row_generator_1 = flow.add_stage('Row Generator', 'Row_Generator_1')


    def get_middle_codegen():
        sf = project.create_subflow('middle_codegen')

        # Stages
        entry_node_2 = sf.add_entry_node('Entry node_2')
        peek_1 = sf.add_stage('Peek', 'Peek_1')

        peek_1.configuration.runtime_column_propagation = False
        exit_node_2 = sf.add_exit_node('Exit node_2')

        # Graph
        link_2_2 = entry_node_2.connect_output_to(peek_1)
        link_2_2.name = 'Link_2'
        entry_node_2_schema = link_2_2.create_schema()
        entry_node_2_schema.add_field('CHAR', 'COLUMN_1').length(100)

        link_3_2 = peek_1.connect_output_to(exit_node_2)
        link_3_2.name = 'Link_3'
        peek_1_schema = link_3_2.create_schema()
        peek_1_schema.add_field('CHAR', 'COLUMN_1').source('Link_2.COLUMN_1').length(100)

        project.update_subflow(sf)

        return sf


    middle_codegen = flow.add_subflow(get_middle_codegen(), 'middle_codegen_1')

    peek_2 = flow.add_stage('Peek', 'Peek_2')


    # Graph
    link_2 = row_generator_1.connect_output_to(middle_codegen).map_to_link('Link_2')
    link_2.name = 'Link_2'
    row_generator_1_schema = link_2.create_schema()
    row_generator_1_schema.add_field('CHAR', 'COLUMN_1').length(100)


    link_3 = middle_codegen.connect_output_to(peek_2).map_from_link('Link_3')
    link_3.name = 'Link_3'
    middle_codegen_1_schema = link_3.create_schema()
    middle_codegen_1_schema.add_field('CHAR', 'COLUMN_1').source('Link_2.COLUMN_1').length(100)


    project.update_flow(flow)

.. skip: end

.. _preparing_data__subflows__using_parameter_sets_in_subflows:

Using Parameter Sets in Subflows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section covers the use of parameter sets in external subflows. Local subflows use the parameter sets and local parameters of their parent.

.. _preparing_data__subflows__using_parameter_sets_in_subflows__using_parameter_sets:

Using Parameter Sets
^^^^^^^^^^^^^^^^^^^^

The process of using parameter sets in subflows is the exact same as using one in a flow. This process is explained in the :ref:`Parameter Sets <preparing_data__parameter_sets>` section. In order for a subflow to use a parameter set its parent must also use the parameter set.

.. _preparing_data__subflows__using_parameter_sets_in_subflows__using_local_parameters:

Using Local Parameters
^^^^^^^^^^^^^^^^^^^^^^

The process of using local parameters in subflows requires a few extra steps compared to how they are used in flows. To add a local parameter, use :py:meth:`Subflow.add_local_parameter <ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow.add_local_parameter>`.
Just like in a flow this method requires a ``parameter_type`` and a ``name``. Optionally, a ``value`` and ``description`` can be provided. ``Value`` can either be a string or an int depending on the ``parameter_type``. This method should be called during the process of creating the subflow.

.. code-block:: python

    def get_created_subflow():
        sf = project.create_subflow('paramset_subflow')

        # Stages
        entry_node_2 = sf.add_entry_node('Entry node_2')
        peek_1 = sf.add_stage('Peek', 'Peek_1')

        peek_1.configuration.runtime_column_propagation = False
        exit_node_2 = sf.add_exit_node('Exit node_2')

        # Graph
        link_2_2 = entry_node_2.connect_output_to(peek_1)
        link_2_2.name = 'Link_2'
        entry_node_2_schema = link_2_2.create_schema()
        entry_node_2_schema.add_field('CHAR', 'COLUMN_1').length(100)

        link_3_2 = peek_1.connect_output_to(exit_node_2)
        link_3_2.name = 'Link_3'
        peek_1_schema = link_3_2.create_schema()
        peek_1_schema.add_field('CHAR', 'COLUMN_1').source('Link_2.COLUMN_1').length(100)

        subflow.add_local_parameter(parameter_type='string', name='make', value='toyota', description='')
        subflow.add_local_parameter('integer', 'num', 13)

        project.update_subflow(sf)

        return sf

Since an external subflow can be reused you may want to set different values for different cases. To do this call the :py:meth:`Subflow.set_local_parameter <ibm_watsonx_data_integration.services.datastage.models.flow.subflow.Subflow.set_local_parameter>` method after adding the subflow to a flow.
This method takes the ``name`` of the local parameter you want to change and the ``value`` you want to set it to.

.. code-block:: python

    paramset_subflow = flow.add_subflow(get_created_subflow(), 'paramset_subflow')

    paramset_subflow.set_local_parameter(name='num', value=25)
    paramset_subflow.set_local_parameter(name='make', value='honda')

    paramset_subflow2 = flow.add_subflow(get_created_subflow(), 'paramset_subflow2')

    paramset_subflow.set_local_parameter(name='num', value=36)
