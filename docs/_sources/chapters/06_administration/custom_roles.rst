.. _administration__custom_roles:

Custom Roles
============
|

Many services map different sets of actions to different platform or service roles.
In some cases, you may want to combine multiple roles' actions to meet your custom use case.
With a custom role you can pick and choose the actions required for your use case, making it easier to grant the right access level to a particular service rather than having to assign multiple roles to accomplish the same task.
The SDK provides functionality to interact with the roles on the watsonx.data integration platform.

This includes operations such as:
    * Listing roles
    * Creating custom roles
    * Updating custom roles
    * Deleting custom roles

Listing roles
~~~~~~~~~~~~~

To retrieve system, service and custom roles you can use the :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.

.. code-block:: python

    >>> platform.roles
    [
        Role(display_name='Service Configuration Reader', role_type='system_role', actions=['global-search-tagging.resource.read', 'resource-controller.instance.retrieve']),
        Role(display_name='Custom role', role_type='custom_role', actions=['iam-groups.groups.update'])
    ]

You can filter roles by using :py:meth:`roles.get_all() <ibm_watsonx_data_integration.platform.Platform.roles.get_all>` method and providing any of the following arguments:
    * ``role_id``: The role id.
    * ``service_name``: Name of the IAM enabled service.
    * ``source_service_name``: Name of the source IAM enabled service.
    * ``policy_type``: Type of the policy.
    * ``service_group_id``: Id of the service group.

.. code-block:: python

    >>> platform.roles.get_all(service_name="cloudshell")
    [
        Role(display_name='Service Configuration Reader', role_type='system_role', actions=['cloudshell.config.read', 'global-search-tagging.resource.read', 'resource-controller.instance.retrieve']),
        Role(display_name='File Manager', role_type='service_role', actions=['cloudshell.server.create', 'cloudshell.server.manage-file'])
    ]

Creating custom roles
~~~~~~~~~~~~~~~~~~~~~

Any number of actions for a single service can be mapped to the new role, but there must be at least one service-defined action to successfully create the new role.

In the UI, adding a new custom role can be done by going to **Manage -> Access (IAM) -> Manage access -> Create**.
You can set a ``Name``, ``Id``, ``Description``, ``Service`` and ``Actions`` of your custom role.

.. image:: ../../_static/images/custom_roles/create_custom_role_button.png
   :alt: Screenshot of the Role creation in the UI
   :align: center
   :width: 100%

.. image:: ../../_static/images/custom_roles/create_custom_role.png
   :alt: Screenshot of the Role creation form in the UI
   :align: center
   :width: 100%

To create a new custom role using the SDK, you can call :py:meth:`Platform.create_role() <ibm_watsonx_data_integration.platform.Platform.create_role>` method with ``name``, ``display_name``, ``service_name`` and ``actions`` parameters. Additionally, you can provide an optional ``description`` parameter.
The ``name`` parameter must start with a capital letter, be no longer than 30 characters, and must not contain spaces.
For more information about roles and actions see `IAM roles and actions <https://cloud.ibm.com/docs/account?topic=account-iam-service-roles-actions>`_.

.. code-block:: python

    >>> platform.create_role(name="CustomRole",
    ...                      service_name="iam-groups",
    ...                      display_name="custom role",
    ...                      actions=["iam-groups.groups.update"])
    Role(display_name='custom role', role_type='custom_role', actions=['iam-groups.groups.update'])

.. note::
    This method may raise a ``requests.exceptions.HTTPError`` with status code 409 if a role with the same ``name`` or ``display_name`` or identical set of ``actions`` already exists.

Updating custom roles
~~~~~~~~~~~~~~~~~~~~~

In the UI, you can update a custom role by clicking three dots next to it and then selecting **Edit**.

.. image:: ../../_static/images/custom_roles/update_custom_role.png
   :alt: Screenshot of the Role update button in the UI
   :align: center
   :width: 100%

To update a role using the SDK, first retrieve it by using the :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.
Next, make in-memory changes to the object and pass it into the :py:meth:`Platform.update_role() <ibm_watsonx_data_integration.platform.Platform.update_role>` method.

.. code-block:: python

    >>> roles=platform.roles
    >>> role_to_update=roles.get(display_name="custom role")
    >>> role_to_update
    Role(display_name='custom role', role_type='custom_role', actions=['iam-groups.groups.update'])
    >>> role_to_update.display_name="updated display name"
    >>> role_to_update.description="updated description"
    >>> role_to_update.actions=["iam-groups.groups.create"]
    >>> role_to_update
    Role(display_name='updated display name', role_type='custom_role', actions=['iam-groups.groups.create'])
    >>> platform.update_role(role_to_update)
    <Response [200]>

.. note::
    You can modify ``display_name``, ``description`` or the ``actions`` that are mapped to the role. The ``name``, ``account_id`` and ``service_name`` can't be changed.

.. note::
    This method may raise a ``requests.exceptions.HTTPError`` with status code 409 if a role with either the same ``display_name`` or identical set of ``actions`` already exists.


Deleting custom roles
~~~~~~~~~~~~~~~~~~~~~

In the UI you can delete a custom role by clicking three dots next to it and then selecting **Delete**.

.. image:: ../../_static/images/custom_roles/delete_custom_role.png
   :alt: Screenshot of the Role deletion button in the UI
   :align: center
   :width: 100%

To delete a role using the SDK, first retrieve it by using :py:attr:`Platform.roles <ibm_watsonx_data_integration.platform.Platform.roles>` property.
Then, pass the object into the :py:meth:`Platform.delete_role() <ibm_watsonx_data_integration.platform.Platform.delete_role>` method.

.. code-block:: python

    >>> role_to_delete = roles.get(display_name='custom role')
    >>> platform.delete_role(role_to_delete)
    <Response [204]>
