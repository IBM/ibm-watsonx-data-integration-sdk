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

Access groups can be retrieved by using the :py:attr:`Platform.access_groups <ibm_watsonx_data_integration.platform.Platform.access_groups>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroups` object, which is a collection of :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` objects.

.. code-block:: python

    >>> access_groups = platform.access_groups
    >>> access_groups
    [
        AccessGroup(name='Developers', description='access group for Developers'),
        AccessGroup(name='Managers', description='Group for managers'),
        AccessGroup(name='New access group', description='the newest description'),
        AccessGroup(name='Public Access', description='This group includes all users and service IDs by default. All group members, including unauthenticated users, are given public access to any resources that are defined in the policies for the group.'),
        AccessGroup(name='Second Test Group', description='Updated Dummy Description'),
        AccessGroup(name='Test access group', description='Dummy access group')
    ]


.. _administration_access_groups__get__access_group:

Get an Access Group
~~~~~~~~~~~~~~~~~~~

To get a specific access group using the SDK, provide the ``access_group_id`` as a filter to the :py:attr:`Platform.access_groups <ibm_watsonx_data_integration.platform.Platform.access_groups>` by using the :py:meth:`AccessGroups.get() <ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroups.get>` function.
This function will return a :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` object if the requested access group was found.

.. code-block:: python

   >>> access_groups = platform.access_groups
   >>> access_groups.get(access_group_id="AccessGroupId-d9a16887-8e76-4478-bb00-35534fdd91e2")

   AccessGroup(name='Test Access Group' description='Dummy Access Group')

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


.. _administration__access_groups__get_members:

Get Member of an Access Group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can view membership information of an Access Group in the IBM Cloud UI by clicking on the desired access group's name. This will load a new page with multiple tabs. The ``Users`` tab lists IBM Cloud Users with membership to the selected access group.
The ``Service ID`` tab lists  Service IDs with membership to the selected access group. The ``Trusted Profiles`` tab lists Trusted Profiles with membership to the selected access group.


In the SDK, all current members of the selected access group can be retrieved by calling the :py:meth:`AccessGroup.get_access_group_members() <ibm_watsonx_data_integration.cpd_models.AccessGroup.get_access_group_members>` function.
This function outputs a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, and :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID` objects that correspond to the members in the selected access group.

.. code-block:: python

   >>> developers_ag.get_access_group_members()
   [ServiceID(id='ServiceId-b6820268-5f8b-4d9b-b46e-1b99d05dea2f', name='Fifth Test Service ID')]


.. _administration__access_groups__add_member:

Add Member(s) to Access Group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To add a member to an Access Group in the UI, you must select the desired access group, navigate to the tab for the type of member you would like to add (one of ``Users``, ``Service ID``, or ``Trusted Profile``),
and add the details of the new member to assign them membership to the selected Access Group.

.. image:: ../../_static/images/access_groups/access_group_home.png
   :alt: Screenshot of selecting an Access Group
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/add_members_button.png
   :alt: Screenshot of clicking on add members button
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/add_members_2.png
   :alt: Screenshot of adding multiple members to an Access Group
   :align: center
   :width: 100%

In the SDK, to add a member to the desired access group, either pass an individual member or a list of members (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`) to the
:py:meth:`AccessGroup.add_members_to_access_group() <ibm_watsonx_data_integration.cpd_models.AccessGroup.add_members_to_access_group>` function. This will add the specified member(s) to the desired Access Group and returns an API response, whose status code should be ``<200>`` if the member(s) were successfully added.

.. code-block:: python

   >>> user = platform.users.get(iam_id="IBMid-6XXXXXXXXP")
   >>> developers_access_group.add_members_to_access_group(user)
   {'members': [{'iam_id': 'IBMid-XXXXXXXXXX', 'type': 'user', 'created_at': '2025-08-01T09:27:12Z', 'created_by_id': 'IBMid-XXXXXXXXXX', 'status_code': 200}]}


.. _administration__access_groups__remove_member:

Remove Member(s) from Access Group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To remove a member from an Access Group in the UI, you must select the desired access group, navigate to the tab for the type of member you would like to remove (one of ``Users``, ``Service ID``, or ``Trusted Profile``),
select the member(s) to remove, and then click the remove button in the top right corner. Confirm the removal of the member(s) on the following popup.

.. image:: ../../_static/images/access_groups/select_members_to_remove.png
   :alt: Screenshot of selecting members to remove
   :align: center
   :width: 100%

In the SDK, to remove a member from the desired access group, either pass an individual member of a list of members (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`) to the
:py:meth:`AccessGroup.remove_members_from_access_group() <ibm_watsonx_data_integration.cpd_models.AccessGroup.remove_members_from_access_group>` function. This will remove the specified member(s) from the desired Access Group and return an API response, whose status code should be ``<204>`` if the member(s) were successfully removed.

.. code-block:: python

   >>> user = platform.users.get(iam_id="IBMid-6XXXXXXXXP")
   >>> developers_access_group.remove_members_from_access_group(user)
   {'access_group_id': 'AccessGroupId-5f54b267-927f-442a-97fd-6ed4581d6998', 'members': [{'iam_id': 'IBMid-6XXXXXXXX0', 'status_code': 204}]}


.. _administration__access_groups__check_membership:

Check Membership to an Access Group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To check the membership of a certain member (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`),
clicking on the desired access group will display all members of each member type with membership to the Access Group.

.. image:: ../../_static/images/access_groups/developers_access_group_home.png
   :alt: Screenshot of landing page for Developers access group
   :align: center
   :width: 100%

In the SDK, to check if an individual member (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`) possesses membership to an Access Group,
pass the member to the :py:meth:`AccessGroup.check_membership() <ibm_watsonx_data_integration.cpd_models.AccessGroup.check_membership>` function. This will return an API response, whose status should be ``<204>`` if the member does posses membership to the specified access group and ``<404>`` if it does not possess membership.

.. code-block:: python

   >>> user = platform.users.get(iam_id="IBMid-6XXXXXXXXP")
   >>> developers_access_group.check_membership(user)
   <Response [204]>


.. _administration__access_groups__add_member_to_multiple_access_groups:

Add Member to Multiple Access Groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To add a member to multiple access groups in the UI, once on the IAM landing page you must navigate to the desired member type (one of ``Users``, ``Service ID``, or ``Trusted Profile``). Once the member type has been selected, ``Service ID`` for this example, select the desired member.
On the following page, navigate to **Assign Group**, then select the groups to add the member to.


.. image:: ../../_static/images/access_groups/iam_home_page.png
   :alt: Screenshot of IAM Landing page with ServiceID circled
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/Fifth_service_ID_circled.png
   :alt: Screenshot of ServiceID page with a ServiceID selected
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/assign_group_service_id.png
   :alt: Screenshot of selected ServiceID page with Assign Groups button circled
   :align: center
   :width: 100%

.. image:: ../../_static/images/access_groups/add_additional_groups_membership.png
   :alt: Screenshot of additional groups added for selected ServiceID
   :align: center
   :width: 100%

In the SDK, to add a member to multiple groups, pass the member (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`) to
the :py:meth:`Platform.add_member_to_multiple_access_groups() <ibm_watsonx_data_integration.platform.Platform.add_member_to_multiple_access_groups>` method along with a list of :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model.AccessGroup` objects which represent the Access Groups the member should be added to.
This will return an API response, whose status should be ``<200>`` for each access group the member was successfully added to.

.. code-block:: python

   >>> user = platform.users.get(iam_id="IBMid-6XXXXXXXXP")
   >>> access_groups = [platform.access_groups.get(access_group_id="AccessGroupId-5f54b267-927f-442a-97fd-6ed4581d6998"), platform.access_groups.get(access_group_id="AccessGroupId-f31bb98a-a2c3-4281-8d54-073a129d071f")]
   >>> platform.add_member_to_multiple_access_groups(user, access_groups)
   {'groups': [{'access_group_id': 'AccessGroupId-5f54b267-927f-442a-97fd-6ed4581d6998', 'status_code': 200}, {'access_group_id': 'AccessGroupId-f31bb98a-a2c3-4281-8d54-073a129d071f', 'status_code': 200}]}


.. _administration__access_groups__remove_member_from_all_access_groups:

Remove Member from All Access Groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To remove a member from all access groups in the SDK, (of types :py:class:`~ibm_watsonx_data_integration.cpd_models.user_model.UserProfile`, :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile`, or :py:class:`~ibm_watsonx_data_integration.cpd_models.service_id_model.ServiceID`) to
the :py:meth:`Platform.remove_member_from_all_access_groups() <ibm_watsonx_data_integration.platform.Platform.remove_member_from_all_access_groups>` method. This will return an API response, whose status should be ``<204>`` for each access group the user was successfully removed from in the account.

.. code-block:: python

   >>> user = platform.users.get(iam_id="IBMid-6XXXXXXXXP")
   >>> platform.remove_member_from_all_access_groups(user)
   {'iam_id': 'IBMid-6XXXXXXXXP', 'groups': [{'access_group_id': 'AccessGroupId-5f54b267-927f-442a-97fd-6ed4581d6998', 'status_code': 204}, {'access_group_id': 'AccessGroupId-7b0be36f-dbea-42d1-9c8e-e962796a30f7', 'status_code': 204}]}
