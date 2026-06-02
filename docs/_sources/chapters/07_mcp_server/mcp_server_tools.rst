.. _mcp_server__tools:

MCP Server Tools
================

The ``ibm_watsonx_data_integration_mcp`` server provides several tools that enable AI assistants to interact with IBM watsonx.data integration workflows. These tools are automatically used by your AI assistant when needed - you don't need to call them manually.

Tools Overview
--------------

- **list_docs** - Lists all available documentation URIs
- **read_doc** - Reads documentation content by URI
- **execute_script** - Executes Python scripts in a separate subprocess
- **list_available_streaming_stages** - Returns available stage labels for streaming flows
- **list_all_available_stage_configurations_streaming** - Lists stage configurations for streaming flows

Available Tools
---------------

list_docs
~~~~~~~~~

Lists all available documentation URIs in the SDK, organized by topic including welcome, overview, getting started, projects, data preparation, access control, and API reference.

read_doc
~~~~~~~~

Reads documentation content by URI. For large documents, the AI assistant can provide a query parameter to retrieve only relevant sections instead of the entire document.

**Parameters**:

- ``uri``: The documentation URI to read
- ``query`` (optional): Search query to filter content in large documents

execute_script
~~~~~~~~~~~~~~

Executes Python scripts in a separate subprocess with a 30-second timeout. The AI assistant uses this to run SDK code on your behalf, so you don't have to copy and paste code into your own environment.

**Parameters**:

- ``code``: The Python script to execute

list_available_streaming_stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns all available stage labels for a streaming flow, including the stage type (origin, processor, destination, executor) and library name. This requires an engine to be installed in the environment.

**Parameters**:

- ``project_id``: The project ID
- ``flow_id``: The flow ID
- ``base_api_url`` (optional): The API URL (default: ``https://api.ca-tor.dai.cloud.ibm.com``)
- ``base_url`` (optional): The base authentication URL (default: ``https://cloud.ibm.com``)

list_all_available_stage_configurations_streaming
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Lists all available stage configurations for a streaming flow, including configuration fields, accepted values, defaults, and whether fields are required or optional. This helps understand what configuration options are available for each stage.

**Parameters**:

- ``project_id``: The project ID
- ``flow_id``: The flow ID
- ``base_api_url`` (optional): The API URL (default: ``https://api.ca-tor.dai.cloud.ibm.com``)
- ``stage_labels`` (optional): List of specific stage labels to get configurations for
- ``base_url`` (optional): The base authentication URL (default: ``https://cloud.ibm.com``)
