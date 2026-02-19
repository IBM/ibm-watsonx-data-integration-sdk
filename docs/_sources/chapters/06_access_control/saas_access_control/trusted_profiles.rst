.. _access_control__trusted_profiles:

Trusted Profiles
================

A trusted profile is a security feature that allows you to grant access to IBM Cloud resources without using the
root account credentials or permanent API keys. Trusted profile management is a platform level service in IBM Cloud
that enables you to manage trusted profiles under your account. The SDK provides functionality to interact with the
trusted profile API.

.. note::
    The SDK currently only supports retrieving trusted profiles.

.. _access_control__trusted_profiles__listing_trusted_profiles:

Listing Trusted Profiles
~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can view all trusted profiles in the current account by **Top left hamburger menu -> Administration -> Access (IAM) -> Trusted Profiles**.

.. image:: /_static/images/trusted_profiles/trusted_profiles.png
   :alt: Screenshot of viewing Trusted Profiles
   :align: center
   :width: 100%


Trusted profiles can be retrieved using the :py:attr:`Platform.trusted_profiles <ibm_watsonx_data_integration.platform.Platform.trusted_profiles>` property.
This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.trusted_profile_model.TrustedProfile` object.

.. skip: start 'there are no trusted profiles right now'

.. code-block:: python

    >>> platform.trusted_profiles.get_all()
    [...TrustedProfile(...)...]

.. skip: end
