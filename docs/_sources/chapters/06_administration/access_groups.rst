.. _administration__access_groups:

Access Groups
=============

An access group is a resource that can be used for assigning policies/roles to a group of members (Users, Trusted Profile, Service IDs).
Access group management is a platform level service in IBM Cloud that enables you to manage access groups under your account.
The SDK provides functionality to interact with the access group API.

This includes operations such as:
    * Listing all access groups under the current Account
    * Creating a new access group
    * Updating an existing access group
    * Deleting an existing access group

.. _administration__access_groups__listing__access_groups:

Listing all Access Groups
~~~~~~~~~~~~~~~~~~~~~~~~~

In the IBM Cloud UI, you can view a list of all access groups under the current account by navigating **Manage -> Access (IAM) -> Access Groups**.

.. image:: ../../_static/images/access_groups/list_access_groups.png
   :alt: Screenshot of the access groups list in the IBM Cloud UI
   :align: center
   :width: 100%

Access groups can be listed by using the :py:attr:`Platform.access_groups <ibm_watsonx_data_integration.platform.Platform.access_groups>` property.
This property returns a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` objects.

.. code-block:: python

    >>> access_group_list = platform.access_groups
    >>> access_group_list
    [
        AccessGroup(name='Developers', description='access group for Developers'),
        AccessGroup(name='Managers', description='Group for managers'),
        AccessGroup(name='New access group', description='the newest description'),
        AccessGroup(name='Public Access', description='This group includes all users and service IDs by default. All group members, including unauthenticated users, are given public access to any resources that are defined in the policies for the group.'),
        AccessGroup(name='Second Test Group', description='Updated Dummy Description'),
        AccessGroup(name='Test access group', description='Dummy access group')
    ]

.. _administration__access_groups__create__an_access_group:

Create an Access Group
~~~~~~~~~~~~~~~~~~~~~~

In the IBM Cloud UI, you can create a new access group by clicking the blue ``Create +`` button on the top right of the pane.
The two fields to be filled in are the ``Name`` and ``Description`` fields.

.. image:: ../../_static/images/access_groups/create_button.png
   :alt: Screenshot of the button to create an access group in the IBM Cloud UI
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/create_access_group.png
   :alt: Screenshot of creating an access group in the IBM Cloud UI
   :align: center
   :width: 100%

An access group can be created by passing in a ``name`` and ``description`` to the :py:meth:`Platform.create_access_group() <ibm_watsonx_data_integration.platform.Platform.create_access_group>` method.
This method returns a newly minted :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` object.


.. code-block:: python

    >>> developers_access_group = platform.create_access_group(name="Developers", description="access group for Developers")
    >>> developers_access_group
    AccessGroup(name='Developers', description='access group for Developers')

.. _administration__access_groups__update__an_access_group:

Update an Access Group
~~~~~~~~~~~~~~~~~~~~~~

In the IBM Cloud UI, you can update an existing access group by clicking on the name of the access group you would like to update.
That opens up a new page with more details about the members, policies, and rules for the access group.
You then have to navigate to **Actions -> Edit** to open a popup pane. There you have to provide the updated ``Name`` or ``Description`` fields.

.. image:: ../../_static/images/access_groups/edit_access_group.png
   :alt: Screenshot of edit button for an access group
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/update_access_group.png
   :alt: Screenshot of UI pane to update an access group
   :align: center
   :width: 100%

An access group can be updated by first updating the ``name`` and ``description`` fields of an existing :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` object.
Following that, the updated :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` object must be passed to the :py:meth:`Platform.update_access_group() <ibm_watsonx_data_integration.platform.Platform.update_access_group>` method.
This method returns an API response, whose status code should be ``<200>`` if the access group was successfully updated.

.. code-block:: python

    >>> developers_ag.name = "New Name"
    >>> developers_ag.description = "New Description"
    >>> platform.update_access_group(developers_ag)
    <Response [200]>

.. _administration__access_groups__delete__an_access_group:

Delete an Access Group
~~~~~~~~~~~~~~~~~~~~~~

In the IBM Cloud UI, you can delete an access group by clicking on the three buttons to the right of the access group you would like to delete.
That opens up a dropdown with a ``Remove`` button. Clicking that button opens a popup that asks you to confirm if you'd like to delete said access group.

.. image:: ../../_static/images/access_groups/remove_access_group.png
   :alt: Screenshot of remove button for an access group
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/delete_access_group.png
   :alt: Screenshot of UI pane to delete an access group
   :align: center
   :width: 100%

An access group can be deleted through the SDK by passing an :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` object to the  :py:meth:`Platform.delete_access_group() <ibm_watsonx_data_integration.platform.Platform.delete_access_group>` method.
This method returns an API response, whose status code should be ``<204>`` if the access group was successfully deleted.

.. code-block:: python

    >>> platform.delete_access_group(developers_ag)
    <Response [204]>
