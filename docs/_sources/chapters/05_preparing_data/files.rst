.. _preparing_data__files:

.. skip: start "our test env has blocked files feature"

Files
=====
|

A file is a ``data asset`` and it is just a regular file where you can store your data.
:py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.ConnectionFile` is a wrapper containing a few more details about a file such as ``hash`` or ``url``.
To download a file with real data you must make an additional method call.

The SDK provides functionality to interact with files.

This includes operations such as:
    * Uploading a file
    * Retrieving file(s)
    * Downloading a file
    * Deleting a file

.. _preparing_data__files__uploading_a_file:

Uploading a File
~~~~~~~~~~~~~~~~

In the UI, you can upload a new File  by navigating to **Overview -> Add data to work with**.

.. image:: ../../_static/images/files/upload_file.png
   :alt: Screenshot of the File uploading in the UI - Step 1
   :align: center
   :width: 100%

.. image:: ../../_static/images/files/upload_file2.png
   :alt: Screenshot of the File uploading in the UI - Step 2
   :align: center
   :width: 100%

In the SDK to upload a new :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.ConnectionFile`,
use the :py:meth:`Platform.upload_file() <ibm_watsonx_data_integration.platform.Platform.upload_file>` method.

You must provide a ``name`` for the new file and a ``file`` path where the file is located on your machine.
:py:meth:`Platform.upload_file() <ibm_watsonx_data_integration.platform.Platform.upload_file>` method returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.ConnectionFile` object.

.. code-block:: python

    >>> file = platform.upload_file(
    ...     name='dummy.txt',
    ...     file=pathlib.Path('/home/me/file.txt'),
    ... )
    ConnectionFile(file_name='dummy.txt')

.. _preparing_data__files__retrieving_an_existing_file:

Retrieving an existing File
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can get Files by navigating to  **Assets -> Data -> Data assets**.

.. image:: ../../_static/images/files/get_files.png
   :alt: Screenshot of the File downloading in the UI
   :align: center
   :width: 100%


In the SDK, Files can be retrieved using :py:class:`Platform.files <ibm_watsonx_data_integration.platform.Platform.files>` property.

This property returns a :py:class:`~ibm_watsonx_data_integration.cpd_models.connections_model.ConnectionFiles` object.

.. code-block:: python

    >>> # Return a list of all files
    >>> files = platform.files
    [ConnectionFile(file_name='dummy.txt')]

.. note::

    Currently :py:class:`Platform.files <ibm_watsonx_data_integration.platform.Platform.files>` does not support any filter arguments.

.. tip::

    For detailed information about parameters and values refer to https://cloud.ibm.com/apidocs/data-ai-common-core#listfiles.


.. _preparing_data__files__downloading_an_existing_file:

Downloading an Existing File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the UI, you can download a File by navigating to **Assets -> Data -> Data assets**.

.. image:: ../../_static/images/files/download_file.png
   :alt: Screenshot of the File downloading in the UI
   :align: center
   :width: 100%

In the SDK file can be downloaded using :py:meth:`Platform.download_file() <ibm_watsonx_data_integration.platform.Platform.download_file>` method.

You must provide a ``file`` object and ``output`` path where the file will be downloaded.

This method returns an HTTP response indicating the status of the download operation.

.. code-block:: python

    >>> file = platform.files.get(name='dummy.txt')
    ConnectionFile(file_name='dummy.txt')
    >>> res = platform.download_file(file=file, output=pathlib.Path('/home/me/download/file.txt'))
    <Response [200]>

.. _preparing_data__files__deleting_a_file:

Deleting a File
~~~~~~~~~~~~~~~

In the UI, you can delete a File by navigating to **Assets -> Data -> Data assets**.

.. image:: ../../_static/images/files/delete_file.png
   :alt: Screenshot of the File deletion in the UI
   :align: center
   :width: 100%

In the SDK pass a ConnectionFile instance to :py:meth:`Platform.delete_file() <ibm_watsonx_data_integration.platform.Platform.delete_file>` method to delete it.

This method returns an HTTP response indicating the status of the delete operation.

.. code-block:: python

    >>> file = platform.files.get(file_name='data.txt')
    ConnectionFile(file_name='data.txt')
    >>> res = platform.delete_file(file)
    <Response [204]>

.. skip: end