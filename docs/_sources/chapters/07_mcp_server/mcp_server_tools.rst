.. _mcp_server__tools:

MCP Server Tools
================

The ``ibm_watsonx_data_integration_mcp`` server provides several tools that enable AI assistants to interact with IBM watsonx.data integration workflows. These tools are automatically used by your AI assistant when needed - you don't need to call them manually.

Tools Overview
--------------

Documentation Tools
~~~~~~~~~~~~~~~~~~~

- **search_sdk_documentation** - Search SDK documentation using semantic similarity
- **get_model_reference** - Look up exact model definitions, field lists, and enum values

Version Tools
~~~~~~~~~~~~~

- **get_mcp_version** - Get the current version of the MCP server
- **get_sdk_version** - Get the version of the IBM watsonx.data Integration SDK

Resource Tools
~~~~~~~~~~~~~~

- **list_resources** - List all available MCP resources
- **read_resource** - Read a specific resource by URI

Execution Tools
~~~~~~~~~~~~~~~

- **execute_script** - Execute generated Python scripts for validation (stdio only, requires auth)

Stage Discovery Tools (Streaming)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **list_available_streaming_stages** - List available streaming stages (stdio only, requires auth)
- **list_all_available_stage_configurations_streaming** - Get streaming stage configurations (stdio only, requires auth)

Stage Discovery Tools (Batch)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **list_available_batch_stages** - List available batch stages (stdio only, requires auth)
- **list_all_available_stage_configurations_batch** - Get batch stage configurations (stdio only, requires auth)

Available Tools
---------------

search_sdk_documentation
~~~~~~~~~~~~~~~~~~~~~~~~

Searches IBM watsonx.data Integration SDK documentation using semantic similarity. This tool uses vector search to find the most relevant documentation sections across all indexed SDK documentation based on a natural language query.

**Parameters**:

- ``query`` (required): Search query describing what to find
- ``top_k`` (optional): Number of top results to return (default: 5, max: 20)

**Returns**: Relevant documentation sections with source information

**Example queries**:

- "How do I authenticate with the platform?"
- "Creating a batch flow with PostgreSQL"
- "Streaming flow configuration options"
- "Job execution and monitoring"

get_model_reference
~~~~~~~~~~~~~~~~~~~

Returns the authoritative field list, enum/Literal values, and method signatures for a named SDK model class. Provides complete field definitions with types and defaults, and strips Pydantic boilerplate methods for clarity. If the class name is not found, provides fuzzy suggestions based on edit distance.

**Parameters**:

- ``class_name`` (required): The exact class name (case-sensitive), e.g. 'ProjectCollaborator', 'JobRun', 'BatchFlow', 'FlowValidationError'

**Returns**: Plain-text reference entry with fields, types, properties, and methods. If class not found, returns error message with suggested similar class names.

**Example class names**:

- ``ProjectCollaborator``
- ``JobRun``
- ``FlowValidationError``
- ``Schedule``

get_mcp_version
~~~~~~~~~~~~~~~

Returns the current version of the MCP server.

**Returns**: Version string

get_sdk_version
~~~~~~~~~~~~~~~

Returns the version of the IBM watsonx.data Integration SDK.

**Returns**: SDK version string

list_resources
~~~~~~~~~~~~~~

Lists all available MCP resources including best practices, intent templates, and skills. Resources are exposed as tools via the ResourcesAsTools transform.

**Returns**: List of available resources with their URIs

read_resource
~~~~~~~~~~~~~

Reads a specific resource by URI. Resources include best practices documentation, intent document templates, and skill guides.

**Parameters**:

- ``uri`` (required): The URI of the resource to read

**Example URIs**:

- ``watsonx://best_practices``
- ``intent://batch-flow``
- ``intent://streaming-flow``
- ``skill://platform/SKILL.md``
- ``skill://project/SKILL.md``
- ``skill://batch-flows/SKILL.md``
- ``skill://streaming-flows/SKILL.md``
- ``skill://jobs-and-job-runs/SKILL.md``

execute_script
~~~~~~~~~~~~~~

Executes generated Python scripts in an isolated subprocess with a 60-second timeout. The script runs with telemetry headers identifying MCP server usage.

**Parameters**:

- ``code`` (required): The Python script to execute

**Returns**: Execution results including return code, stdout, and stderr

**Availability**: Both stdio and SSE transports (can be disabled with ``--disable-execution-tools``)

**Security**: Rejects scripts that access private attributes (patterns matching ``\._[a-zA-Z]``)

list_available_streaming_stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns all available stage information for a streaming flow, including stage type (origin, processor, destination, executor) and library name. This requires an engine to be installed in the environment.

**Parameters**:

- ``project_id`` (required): Project ID
- ``flow_id`` (required): Flow ID
- ``base_api_url`` (optional): The API URL (uses region_name if not provided)
- ``base_url`` (optional): The base authentication URL (default: ``https://cloud.ibm.com``)
- ``region_name`` (optional): Region name from IBMCloudRegion enum (e.g., 'TORONTO', 'DALLAS', 'FRANKFURT', 'LONDON', 'TOKYO', 'SYDNEY')

**Returns**: List of stage information dictionaries with complete stage metadata (excludes config_definitions)

**Availability**: Both stdio and SSE transports (can be disabled with ``--disable-execution-tools``)

list_all_available_stage_configurations_streaming
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns comprehensive configuration information for specified streaming stage types, including all available properties, their types, default values, accepted values, and whether fields are required or optional.

**Parameters**:

- ``project_id`` (required): Project ID
- ``flow_id`` (required): Flow ID
- ``stage_labels`` (required): List of specific stage labels to get configurations for
- ``base_api_url`` (optional): The API URL (uses region_name if not provided)
- ``base_url`` (optional): The base authentication URL (default: ``https://cloud.ibm.com``)
- ``region_name`` (optional): Region name from IBMCloudRegion enum (e.g., 'TORONTO', 'DALLAS', 'FRANKFURT', 'LONDON', 'TOKYO', 'SYDNEY')

**Returns**: Dictionary mapping stage labels to their configuration details, including:

- ``description``: Configuration field description
- ``sdk_config_name``: The actual key used in Stage.configuration
- ``type``: Field type
- ``required``: Whether the field is required
- ``default``: Default value
- ``accepted_values``: List of accepted values (if applicable)

**Availability**: stdio transport only (requires authentication)

list_available_batch_stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns all available stage information for a batch flow. Retrieves available stages from a batch flow's available_stages property and returns complete stage metadata.

**Parameters**:

- ``project_id`` (required): Project ID
- ``flow_id`` (required): Batch flow ID
- ``base_api_url`` (optional): The API URL (uses region_name if not provided)
- ``base_url`` (optional): The base authentication URL (default: ``https://cloud.ibm.com``)
- ``region_name`` (optional): Region name from IBMCloudRegion enum (e.g., 'TORONTO', 'DALLAS', 'FRANKFURT', 'LONDON', 'TOKYO', 'SYDNEY')

**Returns**: List of stage information dictionaries with complete stage metadata

**Availability**: Both stdio and SSE transports (can be disabled with ``--disable-execution-tools``)

list_all_available_stage_configurations_batch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns comprehensive configuration information for specified batch stage types, including all available properties, their types, default values, and enum information. Uses runtime introspection of stage configuration classes to provide accurate, up-to-date information.

**Parameters**:

- ``stage_labels`` (required): List of specific stage labels to get configurations for (e.g., ["Row Generator", "PostgreSQL"])

**Returns**: Dictionary mapping stage labels to their configuration details, including:

- ``configuration_class``: Name of the configuration class
- ``available_properties``: Dictionary of properties with:

  - ``type``: Type annotation string
  - ``current_value``: Current/default value
  - ``value_type``: Python type name
  - ``description``: Field description (if available from Pydantic model)
  - ``enum_values``: Dictionary of enum member names to values (if applicable)
  - ``enum_class``: Full qualified enum class name (if applicable)
  - ``enum_import``: Import path for the enum (if applicable)

**Availability**: Both stdio and SSE transports (can be disabled with ``--disable-execution-tools``)

.. include:: _tool_availability.rst

See :ref:`mcp_server__mcp_server_setup` for more information on configuration and security.
