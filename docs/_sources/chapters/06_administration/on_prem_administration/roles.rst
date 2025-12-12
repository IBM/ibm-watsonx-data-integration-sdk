.. _administration__roles_on_prem:

Roles
=====
|

The SDK provides functionality to interact with roles on on-prem Cloud Pak for Data deployments.

This includes operations such as:
    * Creating roles
    * Listing roles
    * Updating roles
    * Deleting roles


.. _administration__roles_on_prem__creating_roles:

Creating roles
~~~~~~~~~~~~~~

To create a new role using the SDK, you can call :py:meth:`Platform.create_role_on_prem() <ibm_watsonx_data_integration.platform.Platform.create_role_on_prem>` method with ``name``, ``description`` and ``permissions`` parameters.

.. skip: start 'dummy roles values'

.. code-block:: python

    >>> new_role = platform.create_role_on_prem(
    ...     name='MyNewRole',
    ...     description='iam-groups',
    ...     permissions=['create_project', 'manage_locations', 'manage_data_planes', 'view_locations'],
    ... )
    >>> new_role
    RoleOnPrem(role_id='12345' role_name='MyNewRole')

.. skip: end

.. _administration__roles_on_prem__listing_roles:

Listing roles
~~~~~~~~~~~~~

To retrieve roles you can use the :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.

.. skip: start 'dummy roles values'

.. code-block:: python

    >>> platform.roles
    [..., RoleOnPrem(role_id='12345' role_name='MyNewRole')...]

.. skip: end

You can filter roles by using the :py:meth:`get() <ibm_watsonx_data_integration.cpd_models.role_model.Roles.get>` method on the :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property and providing any of the following arguments:
    * ``role_id``: The role id.
    * ``role_name``: Name of the role.

.. skip: start 'dummy roles values'

.. code-block:: python

    >>> platform.roles.get(role_name='MyNewRole')
    RoleOnPrem(role_id='12345' role_name='MyNewRole')

.. skip: end

.. _administration__roles_on_prem__updating_roles:

Updating roles
~~~~~~~~~~~~~~

To update a role using the SDK, first retrieve it by using the :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.
Next, make in-memory changes to the object and pass it into the :py:meth:`Platform.update_role() <ibm_watsonx_data_integration.platform.Platform.update_role>` method.

.. skip: start 'dummy role values'

.. code-block:: python

    >>> new_role.role_name='updated role name'
    >>> new_role.description='updated description'
    >>> new_role.permissions=['administrator']
    >>> platform.update_role(new_role)
    <Response [200]>
    >>> updated_role = platform.roles.get(role_id=new_role.id)
    >>> updated_role
    RoleOnPrem(role_id='12345' role_name='updated role name')

.. skip: end

.. _administration__roles_on_prem__deleting_roles:

Deleting roles
~~~~~~~~~~~~~~

To delete a role using the SDK, first retrieve it by using :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.
Then, pass the object into the :py:meth:`Platform.delete_role() <ibm_watsonx_data_integration.platform.Platform.delete_role>` method.

.. skip: start 'dummy role values'

.. code-block:: python

    >>> platform.delete_role(updated_role)
    <Response [204]>

.. skip: end
