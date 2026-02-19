.. _access_control__access_groups_on_prem:

Access Groups
=============

The SDK provides functionality to interact with the access groups on on-premises Cloud Pak for Data.

This includes operations such as:

    * Creating a new access group
    * Listing access groups
    * Updating an existing access group
    * Deleting an existing access group


.. _access_control__access_groups_on_prem__create__an_access_group:

Create an Access Group
~~~~~~~~~~~~~~~~~~~~~~

An access group can be created by passing in ``name``, ``description`` and ``roles`` to the :py:meth:`Platform.create_access_group_on_prem() <ibm_watsonx_data_integration.platform.Platform.create_access_group_on_prem>` method.
This method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem` object.

.. skip: start 'dummy group values'

.. code-block:: python

    >>> new_group = platform.create_access_group_on_prem(name='Developers', description='access group for Developers', roles=[role1, role2])
    >>> new_group
    AccessGroupOnPrem(group_id='123', name='Developers', members_count=3)

.. skip: end


.. _access_control__access_groups_on_prem__listing__access_groups:

Listing all Access Groups
~~~~~~~~~~~~~~~~~~~~~~~~~

Access groups can be retrieved by using the :py:attr:`Platform.access_groups <ibm_watsonx_data_integration.platform.Platform.access_groups>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupsOnPrem` object, which is a collection of :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem` objects.

.. skip: start 'dummy group values'

.. code-block:: python

    >>> platform.access_groups
    [...AccessGroupOnPrem(group_id='123', name='Developers', members_count=3)...]

.. skip: end

.. _access_control__access_groups_on_prem__update__an_access_group:

Update an Access Group
~~~~~~~~~~~~~~~~~~~~~~

An access group can be updated by first updating the ``name`` and ``description`` fields of an existing :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem` object.
Following that, the updated :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem` object must be passed to the :py:meth:`Platform.update_access_group() <ibm_watsonx_data_integration.platform.Platform.update_access_group>` method.
This method returns an API response, whose status code should be ``200`` if the access group was successfully updated.

.. skip: start 'dummy group values'

.. code-block:: python

    >>> new_group.name = 'New Name'
    >>> new_group.description = 'New Description'
    >>> platform.update_access_group(new_group)
    <Response [200]>

.. skip: end

.. _access_control__access_groups_on_prem__delete__an_access_group:

Delete an Access Group
~~~~~~~~~~~~~~~~~~~~~~

An access group can be deleted through the SDK by passing an :py:class:`~ibm_watsonx_data_integration.cpd_models.access_groups_model_on_prem.AccessGroupOnPrem` object to the  :py:meth:`Platform.delete_access_group() <ibm_watsonx_data_integration.platform.Platform.delete_access_group>` method.
This method returns an API response, whose status code should be ``204`` if the access group was successfully deleted.

.. skip: start 'dummy group values'

.. code-block:: python

    >>> platform.delete_access_group(new_group)
    <Response [204]>

.. skip: end
