.. _mcp_server__skills:

MCP Server Skills
=================

The ``ibm_watsonx_data_integration_mcp`` server provides skill resources that contain domain-specific best practices, workflows, and patterns. These skills are installed server-side and automatically loaded by AI assistants when working on relevant tasks to ensure correct SDK usage.

Skills Overview
---------------

- **platform** - Entry point for all IBM watsonx Data Integration tasks
- **batch-flows** - Building, configuring, and running batch flows
- **streaming-flows** - Building, configuring, and running streaming flows

Available Skills
----------------

platform
~~~~~~~~

**URI**: ``skill://platform/``

**When Used**: This is the entry point skill that AI assistants load first for any IBM watsonx Data Integration task.

**What It Helps With**:

- Understanding the correct startup sequence (reading documentation before generating code)
- Using the right collection methods for projects, flows, jobs, engines, and environments
- Authenticating properly for SaaS and on-premises deployments
- Managing job lifecycles (creating, running, monitoring, and canceling jobs)
- Persisting changes correctly
- Handling errors and configuring error stages

**Key Guidance**: After reading this skill, the AI assistant will load the appropriate specialized skill (batch-flows or streaming-flows) based on your specific task.

batch-flows
~~~~~~~~~~~

**URI**: ``skill://batch-flows/``

**When Used**: AI assistants load this skill when you need to build, configure, or run batch flows.

**What It Helps With**:

- Understanding key differences between batch and streaming flows
- Following the correct workflow from authentication through job execution
- Finding the right stage models and configuration enums
- Working with schemas and column types
- Knowing which of the 150+ available batch stages to use
- Configuring stages correctly with accepted values
- Compiling flows before creating jobs

**Typical Workflow**: Authenticate → Get project → Create/get flow → Add stages → Configure stages → Connect stages → Define schemas → Update flow → Compile flow → Create job → Start job

streaming-flows
~~~~~~~~~~~~~~~

**URI**: ``skill://streaming-flows/``

**When Used**: AI assistants load this skill when you need to build, configure, or run streaming flows.

**What It Helps With**:

- Checking engine health before starting work
- Understanding when environments are needed (optional for streaming)
- Discovering available stages using MCP tools
- Getting stage configurations in manageable batches
- Connecting stages in various patterns (basic, chaining, fan-out, Stream Selector)
- Validating flows before creating jobs
- Working in engineless mode when no engine is available

**Typical Workflow**: Authenticate → Get project → Check engine/environment → Create/get flow → Discover stages → Get configurations → Add stages → Configure stages → Connect stages → Update flow → Validate flow → Create job → Start job

How Skills Work
----------------

Skills are installed server-side and automatically loaded by AI assistants based on the task you're working on. You don't need to install or configure skills manually - the AI assistant will:

1. Start with the **platform** skill to understand general SDK patterns
2. Load the appropriate specialized skill (**batch-flows** or **streaming-flows**) based on your task
3. Use the guidance in these skills to generate correct SDK code
4. Follow the recommended workflows and best practices

This ensures that the AI assistant uses the SDK correctly and follows established patterns, reducing errors and improving code quality.
