.. _projects__access_control:

Access Control
==============

In UI, you can manage access control to a project under the **Manage -> Access Control** tab.
The SDK provides functionality to interact with the access control API.

This includes operations such as:
    * Listing project collaborators
    * Adding project collaborators
    * Updating project collaborators' roles
    * Removing project collaborators

.. _projects__access_control_listing_collaborators:

Listing Collaborators
~~~~~~~~~~~~~~~~~~~~~

In the UI, you can see the list of project collaborators under the **Manage -> Access Control** tab.

.. image:: /_static/images/projects/access_control/access_control.png
   :alt: Screenshot of access control window in a project
   :align: center
   :width: 100%


In the SDK, project collaborators can be retrieved using the :py:attr:`Project.collaborators <ibm_watsonx_data_integration.cpd_models.project_model.Project.collaborators>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.project_collaborator_model.ProjectCollaborators` object.

.. code-block:: python

    >>> project.collaborators
    [...ProjectCollaborator(...)...]

    >>> project.collaborators.get_all()
    [...ProjectCollaborator(...)...]

    >>> collaborator = project.collaborators[0]
    >>> collaborator
    ProjectCollaborator(...)

You can also retrieve a specific project collaborator by using the :py:meth:`ProjectCollaborators.get() <ibm_watsonx_data_integration.common.models.CollectionModel.get>`
method and passing in a ``user_name``.

.. skip: start 'call for non existing username'

.. code-block:: python

    >>> project.collaborators.get(user_name='myuser@ibm.com')
    ProjectCollaborator(user_name='myuser@ibm.com', ...)

.. skip: end

.. _projects__access_control_adding_collaborators:

Adding Collaborators
~~~~~~~~~~~~~~~~~~~~

In the UI, to add a collaborator you can press the **Add collaborators** button, where it will prompt you on which type of collaborator
you would like to add.

.. image:: /_static/images/projects/access_control/add_collaborators_1.png
   :alt: Add collaborator 1
   :align: center
   :width: 100%


There are four types of collaborators you can add to a project:
    * :ref:`User Profiles <administration__users>`
    * :ref:`Access Groups <administration__access_groups>`
    * :ref:`Service IDs <administration__service_ids>` (SaaS only)
    * :ref:`Trusted Profiles <administration__trusted_profiles>` (SaaS only)


Once you select the type of collaborator you would like to add, you can search for the appropriate collaborators to add.

.. image:: /_static/images/projects/access_control/add_collaborators_2.png
   :alt: Add collaborator 2
   :align: center
   :width: 100%

In the SDK, you can add collaborator(s) using either the :py:meth:`Project.add_collaborator() <ibm_watsonx_data_integration.cpd_models.project_model.Project.add_collaborator>`
:py:meth:`Project.add_collaborators() <ibm_watsonx_data_integration.cpd_models.project_model.Project.add_collaborators>` depending on how many collaborators you
would like to add. These functions accept a variety of collaborator types as well the parameter ``role`` which specifies their privileges.

You can add the following type of collaborator types:
    * :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile` or :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model_on_prem.UserProfileOnPrem`
    * :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` or :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem`
    * :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID` (SaaS only)
    * :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile` (SaaS only)

.. skip: start 'cant create custom user from sdk'

.. code-block:: python

    >>> user
    UserProfile(...)

    >>> project.add_collaborator(user, role=CollaboratorRole.ADMIN)
    ProjectCollaborator(user_name='myuser@ibm.com', role='admin', ...)

    >>> access_group
    AccessGroup(access_group_id='access_group_id', ...)

    >>> service_id
    AccessGroup(service_id='service_id', ...)

    >>> project.add_collaborators([access_group, service_id], role=CollaboratorRole.EDITOR)
    [ProjectCollaborator(user_name='access_group_id', role='editor', ...), ProjectCollaborator(user_name='service_id', role='editor', ...)]

.. skip: end

.. _projects__access_control_updating_collaborators:

Updating Project Collaborators' Roles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can edit a collaborator's role by clicking on the edit icon next to their current role. From here, a modal
will appear which will allow you to set the new role for the collaborator.

.. image:: /_static/images/projects/access_control/edit_collaborator.png
   :alt: Screenshot of updating a collaborator's role
   :align: center
   :width: 100%


In the SDK, you can edit collaborator(s) roles by using the :py:attr:`ProjectCollaborator.role <ibm_watsonx_data_integration.cpd_models.project_collaborator_model.ProjectCollaborator.role>`
property and then either using the :py:meth:`Project.update_collaborator() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_collaborator>`
:py:meth:`Project.update_collaborators() <ibm_watsonx_data_integration.cpd_models.project_model.Project.update_collaborators>` method depending on how many
collaborators you would like to update.

.. skip: start 'cant create custom user from sdk'

.. code-block:: python

    >>> collaborator
    ProjectCollaborator(...)

    >>> collaborator.role = ProjectRole.EDITOR
    >>> project.update_collaborator(collaborator)
    ProjectCollaborator(user_name='myuser@ibm.com', role='editor', ...)

    >>> collaborator1
    ProjectCollaborator(user_name='myuser1@ibm.com', role='editor', ...)

    >>> collaborator2
    ProjectCollaborator(user_name='myuser2@ibm.com', role='editor', ...)

    >>> collaborator1.role = ProjectRole.VIEWER
    >>> collaborator2.role = ProjectRole.VIEWER
    >>> project.update_collaborators([collaborator1, collaborator2])
    [ProjectCollaborator(user_name='myuser1@ibm.com', role='viewer', ...), ProjectCollaborator(user_name='myuser2@ibm.com', role='viewer', ...)]

.. skip: end

.. _projects__access_control_removing_collaborators:

Removing Project Collaborators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can remove a collaborator by pressing the remove icon to the right of their entry in the collaborator table.

.. image:: /_static/images/projects/access_control/remove_collaborator.png
   :alt: Screenshot of removing a collaborator
   :align: center
   :width: 100%

In the SDK, you can remove collaborator(s) using either the :py:meth:`Project.remove_collaborator() <ibm_watsonx_data_integration.cpd_models.project_model.Project.remove_collaborator>`
:py:meth:`Project.remove_collaborators() <ibm_watsonx_data_integration.cpd_models.project_model.Project.remove_collaborators>` depending on how many collaborators you
would like to remove.

.. skip: start 'cant create custom user from sdk'

.. code-block:: python

    >>> project.remove_collaborator(collaborator)
    <Response [204]>

    >>> project.remove_collaborators([collaborator1, collaborator2])
    <Response [204]>

.. skip: end
