Tool Availability
-----------------

All tools are available in both stdio and SSE transport modes. However, execution and stage discovery tools can be disabled using the ``--disable-execution-tools`` flag:

.. list-table::
   :header-rows: 1
   :widths: 40 15 45

   * - Tool
     - Requires Auth
     - Can Be Disabled
   * - ``search_sdk_documentation``
     - No
     - No
   * - ``get_model_reference``
     - No
     - No
   * - ``get_mcp_version``
     - No
     - No
   * - ``get_sdk_version``
     - No
     - No
   * - ``list_resources``
     - No
     - No
   * - ``read_resource``
     - No
     - No
   * - ``execute_script``
     - Yes
     - Yes (via ``--disable-execution-tools``)
   * - ``list_available_streaming_stages``
     - Yes
     - Yes (via ``--disable-execution-tools``)
   * - ``list_all_available_stage_configurations_streaming``
     - Yes
     - Yes (via ``--disable-execution-tools``)
   * - ``list_available_batch_stages``
     - Yes
     - Yes (via ``--disable-execution-tools``)
   * - ``list_all_available_stage_configurations_batch``
     - Yes
     - Yes (via ``--disable-execution-tools``)

**Notes:**

- Use ``--disable-execution-tools`` to disable execute_script and stage discovery tools (recommended for remote/untrusted deployments)
- When execution tools are enabled, authentication is required
- When execution tools are disabled, authentication is optional

.. Made with Bob
