.. _mcp_server__mcp_server_setup:

MCP Server Setup
================

This page walks you through every step needed to get the ``data-intg-mcp`` server
running inside your AI coding assistant. The process has two independent parts:

1. **Install** the server binary on your machine (pick one method).
2. **Register** the server with your AI client (each client has its own workflow).

.. contents:: On this page
   :local:
   :depth: 2

----

Requirements
------------

- **Python**: 3.11–3.12 (3.12 recommended)
- **uv** (recommended) *or* any Python 3.11/3.12 virtual environment

----

Step 1 — Install the MCP Server
--------------------------------

Choose **one** of the three methods below. You only need to do this once per machine.

.. _install_uvx:

Method A — ``uvx`` (Recommended, no installation required)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``uvx`` runs the package directly from PyPI without a permanent install. It is the
simplest option and keeps your global Python environment clean.

**Install uv/uvx** (skip if already installed):

.. code-block:: bash

    pip install uv

**Verify the server starts correctly** (downloads all dependencies on first run):

.. code-block:: bash

    uvx --python 3.12 --from ibm_watsonx_data_integration_mcp data-intg-mcp --help

.. _install_uv_tool:

Method B — ``uv tool install`` (Persistent global install)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This installs ``data-intg-mcp`` as a globally available command that your shell can
find at any time, which some clients require.

**Install uv** (skip if already installed):

.. code-block:: bash

    pip install uv

**Install the server:**

.. code-block:: bash

    uv tool install --python 3.12 ibm_watsonx_data_integration_mcp

**Verify:**

.. code-block:: bash

    data-intg-mcp --help

.. _install_pip_venv:

Method C — ``pip`` with a Virtual Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use this method if you prefer traditional virtual environments or cannot use ``uv``.

.. code-block:: bash

    python3.12 -m venv .venv
    source .venv/bin/activate          # Windows: .venv\Scripts\activate
    pip install ibm_watsonx_data_integration_mcp

**Note the full path** to the installed binary — you will need it when configuring
your client:

- **macOS / Linux**: ``<path-to-venv>/bin/data-intg-mcp``
- **Windows**: ``<path-to-venv>\\Scripts\\data-intg-mcp.exe``

**Verify:**

.. code-block:: bash

    data-intg-mcp --help

You should see the following output regardless of which install method you used:

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

----

.. _mcp_config_reference:

Configuration Reference
-----------------------

This section contains ready-to-use JSON blocks for every combination of install
method and authentication type. Copy the block that matches your choices when
following the client-specific instructions in :ref:`Step 2 <step2_client_setup>`.

.. _config_blocks_stdio:

stdio Configurations (local clients)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``stdio`` transport is used by all local desktop clients (Bob, Claude Desktop,
Codex, Copilot). The client launches the server as a child process.

**uvx + SaaS (IBM Cloud API Key)**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

**uvx + On-Premises (CP4D username + password)**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "CP4D_USERNAME": "your-username",
            "CP4D_PASSWORD": "your-password",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false"
          }
        }
      }
    }

**uvx + On-Premises (CP4D username + Zen API Key)**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "CP4D_USERNAME": "your-username",
            "ZEN_API_KEY": "your-zen-api-key",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false"
          }
        }
      }
    }

**uv tool + SaaS**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uv",
          "args": ["tool", "run", "data-intg-mcp"],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

**uv tool + On-Premises (username + password)**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uv",
          "args": ["tool", "run", "data-intg-mcp"],
          "env": {
            "CP4D_USERNAME": "your-username",
            "CP4D_PASSWORD": "your-password",
            "CP4D_URL": "https://your-cp4d-cluster.com",
            "CP4D_DISABLE_SSL_VERIFICATION": "false"
          }
        }
      }
    }

**pip venv + SaaS**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "<path-to-venv>/bin/data-intg-mcp",
          "args": [],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

.. note::
   On Windows replace ``/bin/`` with ``\\Scripts\\`` and append ``.exe`` to the
   command name.

.. _config_blocks_sse:

SSE Configuration (remote/HTTP transport)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use SSE only when the server is running as a standalone HTTP process (e.g., in a
shared team environment). See :ref:`http_sse_transport` for full details.

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp-remote": {
          "url": "http://127.0.0.1:8000/sse"
        }
      }
    }

.. note::
   Environment variables **cannot** be passed from the client config when using
   SSE. Set them in the server's environment before starting the server.

.. _config_windows:

Windows — Required UTF-8 Setting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. important::
   **Windows users must add** ``"PYTHONUTF8": "1"`` **to every** ``env`` **block.**

Windows defaults to cp1252 encoding. Without this setting the server fails on
startup with a ``UnicodeDecodeError`` when reading skill files that contain emoji
and special characters (✅, ❌, →). Add the variable to any configuration block
above:

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "PYTHONUTF8": "1",
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

This applies to all install methods (uvx, uv tool, pip venv).

----

.. _step2_client_setup:

Step 2 — Register with Your AI Client
--------------------------------------

Each AI client has a different mechanism for registering MCP servers. Follow the
section that matches your client.

.. _client_bob:

Bob (VS Code)
~~~~~~~~~~~~~

Bob is the IBM AI coding assistant that runs inside Visual Studio Code. MCP servers
are managed from the **MCP** section of Bob's settings panel.

**Steps:**

1. Open VS Code and click the **Bob** icon in the activity bar.
2. Click the **Settings** (⚙️ gear icon) to open Bob's settings.
3. Scroll to the **MCP** section.
4. Click the **+** (plus) button in the bottom-right of the MCP server list to add a
   new server.
5. Bob will prompt you to select a **configuration scope** — choose either:

   - **Global** — applies to all workspaces on this machine.
   - **Workspace** — applies only to the current project folder.

6. Bob opens the corresponding ``mcp.json`` file for that scope.
7. Add your chosen server entry from :ref:`config_blocks_stdio` into the
   ``"mcpServers"`` object and **save** the file.
8. Click the **↺** (refresh) button next to the **+** to reload the server list.
   The new server should appear immediately — no VS Code restart required.

**Example** ``mcp.json`` **after edit:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

**Verify:** The server name ``data-intg-mcp`` should appear in the MCP server list
in Bob's settings. You can use the **Search MCP servers** box to confirm it is
registered and the toggle next to it is enabled.

.. tip::
   Use the **All** scope dropdown in the MCP panel to filter between global and
   workspace servers if you have both configured.

.. _client_claude:

Claude Desktop
~~~~~~~~~~~~~~

**Steps:**

1. Open Claude Desktop and go to **Claude → Settings → Developer → Edit Config**.
   This opens ``claude_desktop_config.json`` directly in your default text editor.
2. Add your chosen server entry from :ref:`config_blocks_stdio` into the
   ``"mcpServers"`` object and **save** the file.
3. **Quit and restart** Claude Desktop completely (⌘Q / Alt+F4, then reopen).

**Example** ``claude_desktop_config.json`` **after edit:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
          "env": {
            "WATSONX_API_KEY": "YOUR_API_KEY"
          }
        }
      }
    }

.. note::
   If the file already contains a ``"mcpServers"`` key, add the new entry **inside**
   the existing object — do not create a second ``"mcpServers"`` key.

**Verify:** After restarting, open a new Claude conversation. Click the **+** button
next to the message input, then select **Connectors**. ``data-intg-mcp`` should
appear in the connectors list with a toggle to enable it.

.. tip::
   Claude Desktop requires a full restart — toggling the window is not enough.
   Use the system menu or tray to fully quit, then reopen.

.. _client_codex:

Codex (OpenAI)
~~~~~~~~~~~~~~

Codex uses a graphical form to register MCP servers — there is no JSON file to edit.

**Steps:**

1. Open Codex and go to **Settings → Plugins → MCPs** tab.
2. Click **+ Add server**.
3. Fill in the **Connect to a custom MCP** form:

   - **Name**: ``data-intg-mcp``
   - **Type**: ``STDIO``
   - **Command to launch**: ``uvx``
   - **Arguments** (add each one separately using **+ Add argument**):

     .. code-block:: text

         --python
         3.12
         --from
         ibm_watsonx_data_integration_mcp
         data-intg-mcp

   - **Environment variables** (add using **+ Add environment variable**):

     .. list-table::
        :header-rows: 1
        :widths: 40 60

        * - Key
          - Value
        * - ``WATSONX_API_KEY``
          - ``your-ibm-cloud-api-key``

4. Click **Save**.

**Verify:** The server appears in the **Servers** list on the MCPs tab with a toggle
to enable or disable it.

.. note::
   For On-Premises authentication, add ``CP4D_USERNAME``, ``CP4D_PASSWORD`` (or
   ``ZEN_API_KEY``), and ``CP4D_URL`` as additional environment variable rows instead
   of ``WATSONX_API_KEY``. See :ref:`mcp_config_reference` for the full variable list.

.. _client_copilot:

GitHub Copilot in Visual Studio Code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

VS Code supports MCP servers natively from version 1.99 onward.

**Steps:**

1. Open VS Code and open the **GitHub Copilot** panel (``⌃⌘I`` / ``Ctrl+Alt+I``).
2. Click the **Settings** (⚙️ gear icon) in the Copilot panel header.
3. Go to the **MCP Servers** tab.
4. Click the **+** button to add a new server.
5. In the **Command** field that appears, enter the full ``uvx`` launch command:

   .. code-block:: text

       uvx --python 3.12 --from ibm_watsonx_data_integration_mcp data-intg-mcp

6. Confirm — VS Code registers the server but **does not yet let you add environment
   variables** through this form. The server will fail to start until you add your
   credentials in the next step.
7. **Right-click** the newly added ``data-intg-mcp`` entry in the list and choose
   **Show Configuration (JSON)**.
8. VS Code opens the underlying JSON config. Add the ``"env"`` block with your
   credentials and **save**:

   .. code-block:: json

       {
         "servers": {
           "data-intg-mcp": {
             "type": "stdio",
             "command": "uvx",
             "args": [
               "--python", "3.12",
               "--from", "ibm_watsonx_data_integration_mcp",
               "data-intg-mcp"
             ],
             "env": {
               "WATSONX_API_KEY": "YOUR_API_KEY"
             }
           }
         }
       }

9. Right-click the server entry again and choose **Start Server** (or **Restart
   Server** if it attempted to start already).

**Verify:** Switch to **Agent** mode in the Copilot chat and click the
**Select Tools** (🔧) button — the server's tools should be listed and enabled.

.. rubric:: Workspace-level config

For a per-project setup, create ``.vscode/mcp.json`` at the project root with the
same JSON structure shown in step 8 above (the ``"servers"`` key is at the root,
with no outer ``"mcp"`` wrapper).

----

Confirming the Server Started
------------------------------

Regardless of which client you use, the server logs the following line when it has
finished initialising and is ready to accept requests:

.. code-block:: text

    [INFO] Documentation bootstrap complete

If you do not see this message, check the client's MCP logs or developer console for
error output.

----

Authentication
--------------

The server automatically detects which authentication method to use based on the
environment variables present in your configuration.

**Detection priority:**

1. **SaaS** (IAMAuthenticator) — detected when ``WATSONX_API_KEY`` is set.
2. **On-Premises password** (ICP4DAuthenticator) — detected when ``CP4D_USERNAME``
   and ``CP4D_PASSWORD`` are set.
3. **On-Premises Zen API Key** (ZenApiKeyAuthenticator) — detected when
   ``CP4D_USERNAME`` and ``ZEN_API_KEY`` are set.

The server validates credentials on startup and logs which method was detected.

SaaS Authentication (IBM Cloud)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set ``WATSONX_API_KEY`` in the ``env`` block of your client config. See the
configuration blocks in :ref:`config_blocks_stdio` for examples.

.. note::
   For information on how to generate an IBM Cloud API key, see
   :ref:`getting_started_and_tutorials__authentication`.

**Region selection:**

The server defaults to the Toronto region. The AI assistant can ask you to specify
a different region when needed. Available regions (``IBMCloudRegion`` enum):

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - Value
     - Region
   * - ``TORONTO``
     - Toronto, Canada *(default)*
   * - ``DALLAS``
     - Dallas, USA
   * - ``FRANKFURT``
     - Frankfurt, Germany
   * - ``LONDON``
     - London, United Kingdom
   * - ``TOKYO``
     - Tokyo, Japan
   * - ``SYDNEY``
     - Sydney, Australia

The region is passed as a parameter to tools that require it.

On-Premises Authentication (Cloud Pak for Data)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set the CP4D variables in the ``env`` block. Two options are supported:

**Option 1: Username + Password**

.. code-block:: text

    CP4D_USERNAME=your-username
    CP4D_PASSWORD=your-password
    CP4D_URL=https://your-cp4d-cluster.com
    CP4D_DISABLE_SSL_VERIFICATION=false

**Option 2: Username + Zen API Key**

.. code-block:: text

    CP4D_USERNAME=your-username
    ZEN_API_KEY=your-zen-api-key
    CP4D_URL=https://your-cp4d-cluster.com
    CP4D_DISABLE_SSL_VERIFICATION=false

.. note::
   Set ``CP4D_DISABLE_SSL_VERIFICATION`` to ``"true"`` to disable SSL certificate
   verification for self-signed certificates.

Configuring TLS for On-Premises Deployments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your On-Premises cluster uses a custom Certificate Authority (CA), configure TLS
properly instead of disabling SSL verification.

**Step 1 — Log in to your OpenShift cluster:**

.. code-block:: bash

    oc login <CLUSTER_URL>

**Step 2 — Extract the CA certificate:**

.. code-block:: bash

    openssl s_client -showcerts \
      -connect <CLUSTER>:443 </dev/null > ca.crt

Replace ``<CLUSTER>`` with your cluster hostname (e.g., ``your-cp4d-cluster.com``).

**Step 3 — Clean the certificate file:**

Open ``ca.crt`` and remove everything except the certificate blocks:

.. code-block:: text

    -----BEGIN CERTIFICATE-----
    [certificate content]
    -----END CERTIFICATE-----

**Step 4 — Add** ``REQUESTS_CA_BUNDLE`` **to your MCP config:**

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp": {
          "command": "uvx",
          "args": [
            "--python", "3.12",
            "--from", "ibm_watsonx_data_integration_mcp",
            "data-intg-mcp"
          ],
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

Replace ``/path/to/ca.crt`` with the absolute path to your certificate file.

----

Advanced Usage
--------------

Offline Documentation Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your environment cannot reach GitHub Pages, start the server with
``--use-local-docs``. The server reads bundled HTML files, converts them to cached
markdown, and builds a LanceDB index locally.

.. code-block:: bash

    data-intg-mcp --use-local-docs

.. important::
   Bundled docs may lag behind the live site. Re-publish the package periodically
   to keep the bundled copy current.

Cache Management
~~~~~~~~~~~~~~~~

The server caches documentation in a local vector database. Clear the cache when you
need to refresh documentation or troubleshoot retrieval issues:

.. code-block:: bash

    data-intg-mcp --clear-cache

Directories removed:

- ``resources_cache/`` — Cached documentation files
- ``raw_resource_html/`` — Raw HTML files
- ``lance_db/`` — LanceDB vector database

.. _http_sse_transport:

HTTP/SSE Transport
~~~~~~~~~~~~~~~~~~

The default ``stdio`` transport starts the server as a child process of your client.
Use ``sse`` transport to run the server as a standalone HTTP service accessible over
the network:

.. code-block:: bash

    # Basic — no authentication required for doc-only tools
    data-intg-mcp --transport sse --host 0.0.0.0 --port 8000

    # With SDK authentication
    export WATSONX_API_KEY="your-api-key"
    data-intg-mcp --transport sse --host 0.0.0.0 --port 8000

.. include:: _tool_availability.rst

.. important::
   **Security notes for SSE deployments:**

   - Use ``--disable-execution-tools`` to disable execute_script and stage discovery
     tools (strongly recommended for shared or untrusted environments).
   - When execution tools are enabled, authentication is required.
   - When execution tools are disabled, authentication is optional.
   - Environment variables must be set on the server host, not in client config.
   - Apply network-level access controls when exposing the server over HTTP.

Configure a remote MCP client to connect to the SSE server:

.. code-block:: json

    {
      "mcpServers": {
        "data-intg-mcp-remote": {
          "url": "http://127.0.0.1:8000/sse"
        }
      }
    }

**Command-line options reference:**

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--transport``
     - ``stdio`` (default) or ``sse``
   * - ``--host``
     - Host to bind (SSE only, default: ``0.0.0.0``)
   * - ``--port``
     - Port to bind (SSE only, default: ``8000``)
   * - ``--disable-execution-tools``
     - Disable ``execute_script`` and stage discovery tools
   * - ``--clear-cache``
     - Clear all cached documentation before starting
   * - ``--use-local-docs``
     - Use bundled local HTML docs instead of live documentation
