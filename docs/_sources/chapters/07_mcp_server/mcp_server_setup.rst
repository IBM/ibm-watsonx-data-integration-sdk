.. _mcp_server__mcp_server_setup:

MCP Server Setup
================

Requirements
------------

- **Python**: 3.11-3.12 (3.12 recommended)

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

    usage: data-intg-mcp [-h] [--clear-cache] [--disable-execution-tools]
                         [--transport {stdio,sse}] [--host HOST] [--port PORT]
                         [--use-local-docs]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help            show this help message and exit
      --clear-cache         Clear all cached documentation before starting
      --disable-execution-tools
                            Disable execute_script and stage discovery tools
      --transport {stdio,sse}
                            Transport protocol to use (default: stdio)
      --host HOST           Host to bind to when using SSE transport (default: 0.0.0.0)
      --port PORT           Port to bind to when using SSE transport (default: 8000)
      --use-local-docs      Use bundled local HTML docs instead of live documentation

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

    usage: data-intg-mcp [-h] [--clear-cache] [--disable-execution-tools]
                         [--transport {stdio,sse}] [--host HOST] [--port PORT]
                         [--use-local-docs]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help            show this help message and exit
      --clear-cache         Clear all cached documentation before starting
      --disable-execution-tools
                            Disable execute_script and stage discovery tools
      --transport {stdio,sse}
                            Transport protocol to use (default: stdio)
      --host HOST           Host to bind to when using SSE transport (default: 0.0.0.0)
      --port PORT           Port to bind to when using SSE transport (default: 8000)
      --use-local-docs      Use bundled local HTML docs instead of live documentation

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

    usage: data-intg-mcp [-h] [--clear-cache] [--disable-execution-tools]
                         [--transport {stdio,sse}] [--host HOST] [--port PORT]
                         [--use-local-docs]

    MCP Server for IBM watsonx.data Integration

    options:
      -h, --help            show this help message and exit
      --clear-cache         Clear all cached documentation before starting
      --disable-execution-tools
                            Disable execute_script and stage discovery tools
      --transport {stdio,sse}
                            Transport protocol to use (default: stdio)
      --host HOST           Host to bind to when using SSE transport (default: 0.0.0.0)
      --port PORT           Port to bind to when using SSE transport (default: 8000)
      --use-local-docs      Use bundled local HTML docs instead of live documentation

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

Windows UTF-8 Configuration
---------------------------

.. important::
   **Windows users must set the PYTHONUTF8 environment variable** to avoid encoding errors when the server loads skill files.

Due to a limitation in fastmcp 3.2.4, the MCP server requires UTF-8 encoding to properly load skill files that contain
emoji and special characters. On Windows, the default encoding is cp1252, which cannot decode UTF-8 characters like
✅, ❌, and → used in the skill documentation.

**Add this to your MCP configuration on Windows:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": ["--python", "3.12", "--from", "ibm_watsonx_data_integration_mcp", "data-intg-mcp"],
          "env": {
            "PYTHONUTF8": "1",
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

**Why is this needed?**

- Windows defaults to cp1252 encoding, which cannot decode UTF-8 emoji in SKILL.md files
- Setting ``PYTHONUTF8=1`` forces Python to use UTF-8 encoding for all file operations
- Without this setting, the server will fail to start on Windows with a ``UnicodeDecodeError``

**This applies to all installation methods** (uvx, uv tool, pip install) when running on Windows.

Authentication
--------------

The server supports both **SaaS** (IBM Cloud) and **On-Premises** (Cloud Pak for Data) authentication. It automatically detects which authentication method to use based on the environment variables you set.

SaaS Authentication (IBM Cloud)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For SaaS deployments, set the ``WATSONX_API_KEY`` environment variable in the ``env`` block of your MCP client configuration:

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": ["--python", "3.12", "--from", "ibm_watsonx_data_integration_mcp", "data-intg-mcp"],
          "env": {
            "WATSONX_API_KEY": "your-ibm-cloud-api-key"
          }
        }
      }
    }

.. note::
   For information on how to generate an IBM Cloud API key, see :ref:`getting_started_and_tutorials__authentication`.

**Region Selection:**

When using SaaS authentication, the server defaults to the Toronto region. The AI assistant can ask you to specify a different IBM Cloud region when needed. Available regions (from the ``IBMCloudRegion`` enum):

- ``TORONTO`` - Toronto, Canada (default)
- ``DALLAS`` - Dallas, USA
- ``FRANKFURT`` - Frankfurt, Germany
- ``LONDON`` - London, United Kingdom
- ``TOKYO`` - Tokyo, Japan
- ``SYDNEY`` - Sydney, Australia

The region is passed as a parameter to the tools that require it, ensuring you're connecting to the correct regional endpoint.

On-Premises Authentication (Cloud Pak for Data)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For On-Premises deployments, you have two authentication options:

**Option 1: Username + Password**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": ["--python", "3.12", "--from", "ibm_watsonx_data_integration_mcp", "data-intg-mcp"],
          "env": {
            "CP4D_USERNAME": "your-username",
            "CP4D_PASSWORD": "your-password",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false"
          }
        }
      }
    }

**Option 2: Username + Zen API Key**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": ["--python", "3.12", "--from", "ibm_watsonx_data_integration_mcp", "data-intg-mcp"],
          "env": {
            "CP4D_USERNAME": "your-username",
            "ZEN_API_KEY": "your-zen-api-key",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false"
          }
        }
      }
    }

.. note::
   **SSL Verification:**

   - ``CP4D_DISABLE_SSL_VERIFICATION``: Set to ``"true"`` to disable SSL certificate verification for On-Premises deployments (useful for self-signed certificates). Default is ``"false"``.

Configuring TLS for On-Premises Deployments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your On-Premises cluster uses self-signed certificates or custom Certificate Authorities (CA), you may need to configure TLS properly instead of disabling SSL verification. Here's how to set up TLS:

**Step 1: Log in to your OpenShift cluster**

.. code-block:: bash

    oc login <CLUSTER_URL>

**Step 2: Extract the cluster's CA certificate**

.. code-block:: bash

    openssl s_client -showcerts \
      -connect <CLUSTER>:443 </dev/null > ca.crt

Replace ``<CLUSTER>`` with your cluster hostname (e.g., ``your-cp4d-cluster.com``).

**Step 3: Clean up the certificate file**

Open the ``ca.crt`` file and remove all text except the certificate blocks. The file should only contain lines like:

.. code-block:: text

    -----BEGIN CERTIFICATE-----
    [certificate content]
    -----END CERTIFICATE-----

Remove any connection information, verification messages, or other output from the openssl command.

**Step 4: Configure the MCP server to use the CA certificate**

Add the ``REQUESTS_CA_BUNDLE`` environment variable to your MCP server configuration, pointing to the cleaned ``ca.crt`` file:

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": ["--python", "3.12", "--from", "ibm_watsonx_data_integration_mcp", "data-intg-mcp"],
          "env": {
            "CP4D_USERNAME": "your-username",
            "CP4D_PASSWORD": "your-password",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false",
            "REQUESTS_CA_BUNDLE": "/path/to/ca.crt"
          }
        }
      }
    }

Replace ``/path/to/ca.crt`` with the absolute path to your cleaned certificate file.

**Authentication Detection Priority:**

1. SaaS (IAMAuthenticator) - if ``WATSONX_API_KEY`` is set
2. On-Premises (ICP4DAuthenticator) - if ``CP4D_USERNAME`` and ``CP4D_PASSWORD`` are set
3. On-Premises (ZenApiKeyAuthenticator) - if ``CP4D_USERNAME`` and ``ZEN_API_KEY`` are set

The server validates authentication credentials on startup and will display which authentication method was detected.

Advanced Usage
--------------

Offline Documentation Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your environment cannot access GitHub Pages, start the server with ``--use-local-docs``. In this mode, the server reads packaged HTML files from the bundled documentation, converts them into cached markdown files, and then builds the LanceDB index in the normal runtime cache.

Example:

.. code-block:: bash

    data-intg-mcp --use-local-docs

.. important::
   - Offline docs may not be the most up to date because they do not use the live documentation site.
   - Refresh the packaged docs periodically before publishing or deploying if you want the bundled copy to stay current.

Cache Management
~~~~~~~~~~~~~~~~

The server caches documentation and creates a vector database for efficient retrieval. To clear all cached data:

.. code-block:: bash

    data-intg-mcp --clear-cache

This removes:

- ``resources_cache/`` - Cached documentation files
- ``raw_resource_html/`` - Raw HTML files
- ``lance_db/`` - Vector database (LanceDB)

Use this command when you need to refresh the documentation cache or troubleshoot issues with the vector database.

HTTP/SSE Transport
~~~~~~~~~~~~~~~~~~

By default, the MCP server runs over stdio transport (for local MCP clients like Claude Desktop or Bob). You can also run it as an HTTP server using Server-Sent Events (SSE) transport for remote access:

.. code-block:: bash

    # Run as HTTP/SSE server (API key is optional for SSE)
    data-intg-mcp --transport sse --host 0.0.0.0 --port 8000

    # If you need SDK operations that require authentication, set the API key:
    export WATSONX_API_KEY="your-api-key"
    data-intg-mcp --transport sse --host 0.0.0.0 --port 8000

.. include:: _tool_availability.rst

.. important::
   **Security and Authentication Notes:**

   - Use ``--disable-execution-tools`` to disable execute_script and stage discovery tools (recommended for remote/untrusted deployments)
   - When execution tools are enabled, authentication is required (``WATSONX_API_KEY`` or CP4D credentials)
   - When execution tools are disabled, authentication is optional
   - The SSE server allows remote connections, so ensure proper network security measures are in place
   - Environment variables must be set on the server side, not in the client configuration
   - Consider using authentication/authorization mechanisms when exposing the server over HTTP

**MCP Client Configuration for SSE:**

For remote MCP clients, configure the server URL:

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp-remote": {
          "url": "http://127.0.0.1:8000/sse"
        }
      }
    }

.. note::
   Unlike stdio transport, environment variables cannot be passed from the client config when using SSE. They must be set in the server's environment before starting the server.

**Command Line Options:**

- ``--transport``: Choose ``stdio`` (default) or ``sse``
- ``--host``: Host to bind to for SSE (default: 0.0.0.0, ignored for stdio)
- ``--port``: Port to bind to for SSE (default: 8000, ignored for stdio)
- ``--disable-execution-tools``: Disable execute_script and stage discovery tools (recommended for remote deployments)
- ``--clear-cache``: Clear all cached documentation before starting
- ``--use-local-docs``: Use bundled local HTML docs instead of retrieving documentation live
