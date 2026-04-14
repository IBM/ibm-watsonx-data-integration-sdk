.. _welcome__mcp_server:

MCP Server Setup
================

The ``ibm_watsonx_data_integration_mcp`` package is an MCP (Model Context Protocol) server for IBM watsonx.data integration.
It enables AI assistants to interact with your data integration workflows directly, without needing to write SDK code.

Requirements
------------

- **Python**: 3.12

Installation
------------

There are three ways to run the MCP server depending on your preferred toolchain.

Using ``uvx`` (Recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The fastest way to get started (no installation required).

If you don't have ``uvx``, install it first:

.. code-block:: bash

    pip install uv

Verify it works:

.. code-block:: bash

    uvx --python 3.12 --from ibm_watsonx_data_integration_mcp data-intg-mcp --help

**MCP server configuration:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python",
            "3.12",
            "--from",
            "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY" # pragma: allowlist secret
          }
        }
      }
    }

Using ``uv`` as a Tool
~~~~~~~~~~~~~~~~~~~~~~

If you don't have ``uv``, install it first:

.. code-block:: bash

    pip install uv

Install:

.. code-block:: bash

    uv tool install --python 3.12 ibm_watsonx_data_integration_mcp

Verify the installation:

.. code-block:: bash

    data-intg-mcp --help

**MCP server configuration:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uv",
          "args": [
            "tool",
            "run",
            "data-intg-mcp"
          ],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY" # pragma: allowlist secret
          }
        }
      }
    }

Using ``pip`` with a Virtual Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    python3.12 -m venv .venv
    source .venv/bin/activate
    pip install ibm_watsonx_data_integration_mcp

Verify the installation:

.. code-block:: bash

    data-intg-mcp --help

**MCP server configuration:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "<Path to .venv>/bin/data-intg-mcp",
          "args": [],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY" # pragma: allowlist secret
          }
        }
      }
    }

Client Integration
------------------

Claude Desktop
~~~~~~~~~~~~~~

Navigate to **Claude → Settings → Developer → Edit Config** and add one of the MCP server configurations above
to your ``claude_desktop_config.json`` file.

Bob (VS Code)
~~~~~~~~~~~~~

Navigate to **VS Code → Open Bob → Settings (⚙️) → MCP → Open Global MCPs** and add one of the MCP server
configurations above to your Bob MCP config file.

Authentication
--------------

Set your ``WATSONX_API_KEY`` in the ``env`` block of your MCP client configuration. The server validates this key on startup.

.. note::
   You can generate an API key from your IBM Cloud account at: `https://cloud.ibm.com/iam/apikeys <https://cloud.ibm.com/iam/apikeys>`_
