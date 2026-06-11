.. _mcp_server__skills:

MCP Server Skills
=================

The ``ibm_watsonx_data_integration_mcp`` server provides skill resources that contain domain-specific best practices, workflows, and patterns. These skills are installed server-side and automatically loaded by AI assistants when working on relevant tasks to ensure correct SDK usage.

Skills Overview
---------------

The MCP server includes the following skills:

- **platform** - Entry point for all IBM watsonx.data Integration tasks
- **project** - Project creation, listing, and management
- **batch-flows** - Building, configuring, and running batch flows
- **streaming-flows** - Building, configuring, and running streaming flows
- **jobs-and-job-runs** - Job creation, execution, and monitoring

Available Skills
----------------

platform
~~~~~~~~

**URI**: ``skill://platform/SKILL.md``

**Description**: Authentication patterns, Platform object initialization, and core SDK sequencing rules for IBM watsonx.data Integration. This is the entry point skill that should be read first for any data integration task.

**What It Covers**:

- Mandatory startup sequence (reading best practices before any code generation)
- Authentication patterns for SaaS (IAM) and On-Premises (CP4D) deployments
- Platform object initialization
- Core SDK sequencing rules and order of operations
- Collection methods for projects, flows, jobs, engines, and environments
- Job lifecycle management (creating, running, monitoring, and canceling jobs)
- Persisting changes correctly
- Error handling and configuration

**Key Guidance**: This skill emphasizes that code examples are non-authoritative and serve only to illustrate patterns and ordering. All identifiers (methods, fields, enums, types) must be confirmed via ``search_sdk_documentation`` (primary) or ``get_model_reference`` (fallback).

**Typical Usage**: AI assistants load this skill first to understand general SDK patterns, then load the appropriate specialized skill (project, batch-flows, streaming-flows, or jobs-and-job-runs) based on the specific task.

project
~~~~~~~

**URI**: ``skill://project/SKILL.md``

**Description**: Project lifecycle management in IBM watsonx.data Integration - creating, listing, retrieving, and deleting projects.

**What It Covers**:

- Project types (watsonx.data Integration vs Cloud Pak for Data)
- Proper parameter usage (project_id vs id)
- Creating new projects with required and optional parameters
- Listing and filtering projects
- Retrieving specific projects
- Accessing project resources (flows, jobs, engines, environments)
- Deleting projects
- Project collaborator management

batch-flows
~~~~~~~~~~~

**URI**: ``skill://batch-flows/SKILL.md``

**Description**: Complete workflow for creating batch data integration flows including stage configuration, schema creation patterns, link management, and compilation.

**What It Covers**:

- Key differences between batch and streaming flows
- Complete workflow from authentication through job execution
- Finding the right stage models and configuration enums
- Working with schemas and column types
- Available batch stages (150+ stages)
- Stage configuration with accepted values
- Connecting stages with links
- Schema definitions and reuse
- Flow compilation before job creation
- Job creation and execution

streaming-flows
~~~~~~~~~~~~~~~

**URI**: ``skill://streaming-flows/SKILL.md``

**Description**: Complete workflow, stage discovery, configuration, and connection patterns for streaming flows.

**What It Covers**:

- Engine health checks before starting work
- Environment management (optional for streaming)
- Stage discovery using MCP tools
- Getting stage configurations in manageable batches
- Connecting stages in various patterns (basic, chaining, fan-out, Stream Selector)
- Flow validation before job creation
- Working in engineless mode when no engine is available
- Error handling and error stage configuration

jobs-and-job-runs
~~~~~~~~~~~~~~~~~

**URI**: ``skill://jobs-and-job-runs/SKILL.md``

**Description**: Creating, managing, and monitoring jobs and job runs in IBM watsonx.data integration.

**What It Covers**:

- Job creation for batch and streaming flows
- Job configuration and parameters
- Runtime parameters and parameter sets
- Starting and stopping jobs
- Monitoring job execution status
- Checking job logs and metrics
- Job run lifecycle management
- Canceling and deleting jobs
- Setting retention policies
- Scheduling jobs
- Job state management

How Skills Work
----------------

Skills are installed server-side and automatically loaded by AI assistants based on the task you're working on. You don't need to install or configure skills manually - the AI assistant will:

1. Start with the **platform** skill to understand general SDK patterns and mandatory startup sequence
2. Load the appropriate specialized skill (**project**, **batch-flows**, **streaming-flows**, or **jobs-and-job-runs**) based on your task
3. Use the guidance in these skills to generate correct SDK code
4. Follow the recommended workflows and best practices
5. Verify all code against SDK documentation using ``search_sdk_documentation`` and ``get_model_reference`` tools

This ensures that the AI assistant uses the SDK correctly and follows established patterns, reducing errors and improving code quality.

Best Practices Resources
------------------------

In addition to skills, the MCP server provides best practices resources that contain canonical usage rules and patterns:

**watsonx://best_practices**
  Comprehensive best practices for the IBM watsonx.data Integration SDK, including:

  - Core concepts and platform-centric architecture
  - Authentication patterns
  - Configuration management
  - Streaming and batch flow best practices
  - Common invalid patterns to avoid
  - Forbidden patterns (never use underscore-prefixed private APIs)
  - Code generation best practices
  - MCP server usage patterns
  - Quick reference and troubleshooting

Intent Document Resources
--------------------------

The MCP server provides intent document templates that enable a structured, requirements-first approach to flow creation. Intent documents are **MANDATORY** when creating batch or streaming flows - they must be generated and approved before any code is written.

What Are Intent Documents?
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Intent documents are structured specifications that capture all requirements, configurations, and design decisions before generating code for data integration flows. They serve as a contract between you and the AI assistant, ensuring complete understanding of what needs to be built.

An intent document includes:

- **Flow Overview**: Business purpose and high-level description
- **Flow Topology Diagram**: A Mermaid diagram showing all stages and their connections (MANDATORY)
- **Source/Destination Schemas**: Complete data structure definitions
- **Transformation Logic**: Detailed description of data transformations
- **Stage Configurations**: All configuration parameters for each stage
- **Connection Map**: How stages are connected (for streaming flows)
- **Error Handling**: Error handling strategy and configuration
- **Validation Checklist**: Verification steps before code generation

Why Intent Documents Are Mandatory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Intent documents prevent the most common causes of code generation failures:

1. **Incomplete Requirements**: Forces capture of all necessary information upfront
2. **Configuration Errors**: Validates stage configurations against available options before code generation
3. **Schema Mismatches**: Ensures schemas are defined and compatible across all stages
4. **Missing Connections**: Visualizes the complete flow topology to catch missing or incorrect connections
5. **Unclear Transformations**: Documents transformation logic explicitly

**Statistics**: Intent documents prevent 80%+ of code generation errors by catching issues during the planning phase rather than during execution.

Available Intent Resources
~~~~~~~~~~~~~~~~~~~~~~~~~~

``intent://overview``
  Overview of the mandatory intent document generation approach for all flow creation. Explains the workflow, benefits, and requirements.

``intent://batch-flow``
  Template and instructions for batch flow intent documents. Includes:

  - Flow overview and business purpose
  - Complete data source schemas
  - Transformation logic
  - Destination configurations
  - Stage-by-stage configuration details
  - Schema mappings for all links
  - Validation checklist

``intent://streaming-flow``
  Template and instructions for streaming flow intent documents. Includes:

  - Flow overview and business purpose
  - Engine and environment pre-flight check
  - Stage discovery results (from MCP tools)
  - Complete stage configurations with accepted values
  - Connection map for all stage connections
  - Error handling configuration
  - Validation strategy

Workflow for Flow Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The intent document workflow is **NON-NEGOTIABLE** for all flow creation:

1. **Read Intent Resource**: AI assistant reads the appropriate intent resource (``intent://batch-flow`` or ``intent://streaming-flow``)
2. **Generate Intent Document**: AI assistant generates a complete intent document following the template
3. **Include Mermaid Diagram**: Every intent document MUST include a Mermaid diagram showing the complete flow topology (this is MANDATORY)
4. **Present for Review**: AI assistant presents the intent document to you for review and approval
5. **Get Approval**: You review and approve the intent document (or request changes)
6. **Generate Code**: Only after approval, the AI assistant generates and executes the Python code

**The Mermaid diagram is MANDATORY and NON-NEGOTIABLE**. It must show all stages and connections. Failure to follow these steps will result in incorrect code generation.

Benefits of Intent Documents
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Prevents Errors**: Catches configuration and schema issues before code generation
- **Ensures Completeness**: Forces capture of all requirements upfront
- **Provides Documentation**: Creates a reviewable specification of the flow design
- **Enables Better Code**: AI assistants generate more accurate code from complete specifications
- **Saves Time**: Reduces debugging and rework by getting requirements right the first time
- **Facilitates Review**: Provides a clear, visual representation of the flow for stakeholder review

Example Intent Document Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A typical batch flow intent document includes:

.. code-block:: text

    # Batch Flow Intent Document

    ## Flow Overview
    - **Purpose**: Migrate customer data from PostgreSQL to Snowflake
    - **Type**: Batch ETL pipeline
    - **Frequency**: Daily at 2 AM

    ## Flow Topology
    ```mermaid
    graph LR
        A[PostgreSQL Source] --> B[Filter Active Customers]
        B --> C[Transform Data]
        C --> D[Snowflake Destination]
    ```

    ## Source Schema
    - Table: customers
    - Columns: id (int), name (string), email (string), status (string), created_at (timestamp)

    ## Transformation Logic
    - Filter: status = 'active'
    - Transform: Convert created_at to ISO 8601 format
    - Map: id → customer_id, name → full_name

    ## Destination Schema
    - Table: active_customers
    - Columns: customer_id (int), full_name (string), email (string), registration_date (timestamp)

    ## Stage Configurations
    [Detailed configuration for each stage]

    ## Validation Checklist
    - [ ] All source columns mapped
    - [ ] Transformation logic verified
    - [ ] Destination schema matches
    - [ ] Error handling configured

This structured approach ensures that all requirements are captured and validated before any code is generated, resulting in more reliable and maintainable data integration flows.
