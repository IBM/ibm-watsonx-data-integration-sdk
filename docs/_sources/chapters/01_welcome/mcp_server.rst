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

Verify it works (this will install all dependencies and may take a few minutes on first run):

.. code-block:: bash

    uvx --python 3.12 --from ibm_watsonx_data_integration_mcp data-intg-mcp --help

You should see output similar to:

.. code-block:: text

    usage: data-intg-mcp [-h] [--clear-cache]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help     show this help message and exit
      --clear-cache  Clear all cached documentation before starting (resources_cache/ and docs/)

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
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

Using ``uv`` as a Tool
~~~~~~~~~~~~~~~~~~~~~~

If you don't have ``uv``, install it first:

.. code-block:: bash

    pip install uv

Install (this will install all dependencies and may take a few minutes):

.. code-block:: bash

    uv tool install --python 3.12 ibm_watsonx_data_integration_mcp

Verify the installation:

.. code-block:: bash

    data-intg-mcp --help

You should see output similar to:

.. code-block:: text

    usage: data-intg-mcp [-h] [--clear-cache]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help     show this help message and exit
      --clear-cache  Clear all cached documentation before starting (resources_cache/ and docs/)

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
            "WATSONX_API_KEY": "YOUR_API_KEY"
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

This will install the MCP server and all its dependencies into the virtual environment.

Verify the installation:

.. code-block:: bash

    data-intg-mcp --help

You should see output similar to:

.. code-block:: text

    usage: data-intg-mcp [-h] [--clear-cache]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help     show this help message and exit
      --clear-cache  Clear all cached documentation before starting (resources_cache/ and docs/)

**MCP server configuration:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "<Path to .venv>/bin/data-intg-mcp",
          "args": [],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

Client Integration
------------------

Once you've added the MCP server configuration to your client, the server will start automatically when the client launches.
You'll know the server has finished initializing when you see the following message in the logs:

.. code-block:: text

    [INFO] Documentation bootstrap complete

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
   For information on how to generate an IBM Cloud API key, see :ref:`getting_started_and_tutorials__authentication`.
