.. _projects__projects:

Projects
========
|

A project is a collaborative workspace in the watsonx.data integration platform where users can organize assets such as jobs, flows, and environments.

Projects provide an isolated context for managing and grouping related assets, allowing for better organization, access control, and lifecycle management.

The SDK provides functionality to interact with projects on the watsonx.data integration platform.

This includes operations such as:
    * Creating a project
    * Retrieving existing projects
    * Updating a project
    * Deleting a project

.. _projects__projects__creating_a_project:

Creating a Project
~~~~~~~~~~~~~~~~~~

In the UI, you can create a new project by clicking **New project** button.

.. image:: /_static/images/projects/create_project.png
   :alt: Creating a project via the UI
   :align: center
   :width: 100%

|

To create a new :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object using the SDK, use the :py:meth:`Platform.create_project() <ibm_watsonx_data_integration.platform.Platform.create_project>` method.

You must specify the ``name`` attribute.
Additionally, you can provide optional parameters such as ``description``, ``tags``, ``public`` and ``project_type``.
If you do not pass in a ``project_type`` parameter, the SDK will default to ``wx``, effectively creating the project within the ``watsonx`` view.

This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object.

.. code-block:: python

    >>> my_project = platform.create_project(
    ...     name='Test Project',
    ...     description='Test Project Description',
    ...     tags=['flow_test_project'],
    ...     public=True,
    ...     project_type='wx'
    ... )
    >>> my_project
    Project(name='Test Project', ...)

Retrieving Existing Projects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To list existing projects in the UI, navigate to **View all projects**.

.. image:: /_static/images/projects/list_all_projects.png
   :alt: Listing projects via the UI
   :align: center
   :width: 50%

|

Projects can be retrieved using :py:class:`Platform.projects <ibm_watsonx_data_integration.platform.Platform.projects>`.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Projects` object.
You can also further filter and refine the projects returned based on attributes including ``name`` and ``project_id``.

.. code-block:: python

    >>> # Returns a list of all projects
    >>> platform.projects
    [...Project(name='Test Project', ...)...]
    >>> # Returns first project matching given `name`
    >>> platform.projects.get(name='Test Project')
    Project(name='Test Project', ...)
    >>> # Returns a list of all projects that match `name`
    >>> platform.projects.get_all(name='Test Project')
    [Project(name='Test Project', ...)]

Updating a Project
~~~~~~~~~~~~~~~~~~

In the UI, you can update a project by clicking on the pencil icon under the **Manage** tab of the project.
Once you have made the necessary changes, click **Save** to update the project.

.. image:: /_static/images/projects/updating_project.png
   :alt: Updating a project via the UI
   :align: center
   :width: 100%

|

To update a project via the SDK, first make the necessary in-memory changes to your :py:class:`~ibm_watsonx_data_integration.cpd_models.project_model.Project` object.
Next, pass in this object into the :py:meth:`Platform.update_project() <ibm_watsonx_data_integration.platform.Platform.update_project>` method.

This method returns an HTTP response indicating the status of the update operation.

.. code-block:: python

    >>> my_project = platform.projects.get(name='Test Project')
    >>> my_project.name = 'New name'
    >>> my_project.description = 'New description.'
    >>> platform.update_project(my_project)
    <Response [200]>


Deleting a Project
~~~~~~~~~~~~~~~~~~

In the UI, you can delete a project by selecting its title from the projects list.
To delete a project, click the  **Delete** button in the top bar.

.. image:: /_static/images/projects/delete_project.png
   :alt: Deleting a project via the UI
   :align: center
   :width: 100%

|

To delete a project via the SDK, you can pass the project instance to the :py:meth:`Platform.delete_project() <ibm_watsonx_data_integration.platform.Platform.delete_project>` method.
This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> platform.delete_project(my_project)
    <Response [204]>
