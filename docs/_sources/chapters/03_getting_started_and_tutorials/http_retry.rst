HTTP Retry Management
======================
|

.. versionadded:: 1.1.0

The SDK includes a built-in HTTP retry mechanism to handle transient failures in API requests. This includes automatic retry
with exponential backoff, jitter, and configurable retry policies for different HTTP status codes.

Understanding Retry Behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The retry mechanism automatically handles transient failures such as rate limiting (429), server errors (5xx), and network issues.
By default, the SDK retries the following HTTP status codes:

- **429** (Too Many Requests) - Rate limiting
- **500** (Internal Server Error)
- **502** (Bad Gateway)
- **503** (Service Unavailable)
- **504** (Gateway Timeout)

Each retry uses exponential backoff with jitter to avoid overwhelming the server and prevent thundering herd problems.

Default Retry Settings
~~~~~~~~~~~~~~~~~~~~~~

The default retry configuration applies to all HTTP requests unless overridden:

- **Max Attempts:** 3
- **Initial Delay:** 1.0 second
- **Exponential Factor:** 2.0x
- **Jitter:** Enabled (±25% randomness)

Special handling for rate limiting (429):

- **Max Attempts:** 10
- **Initial Delay:** 1.0 second
- **Exponential Factor:** 1.5x
- **Jitter:** Enabled

Viewing Current Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To display the current retry configuration, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.show` method.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import RetryConfig

    RetryConfig.show()

This will output:

.. code-block:: text

    ======================================================================
    RETRY CONFIGURATION
    STATUS: RETRIES ENABLED
    ======================================================================

    Default Settings:
    Applied to status codes: [500, 502, 503, 504]
      Max Attempts:    3
      Max Time:        None (max allowed: 3600s)
      Initial Delay:   1.0s
      Exp Factor:      2.0x
      Jitter:          Enabled

    Status-Specific Settings:
      HTTP 429:
        Max Attempts:  10
        Max Time:      None (max allowed: 3600s)
        Initial Delay: 1.0s
        Exp Factor:    1.5x
        Jitter:        Enabled
    ======================================================================

Configuring Global Retry Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Changing Default Settings
-------------------------

You can change the default retry settings that apply to all retryable status codes using the
:py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.set` method.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import RetryConfig, RetrySettings

    # Change default retry behavior
    RetryConfig.set(
        default=RetrySettings(
            max_attempts=5,
            max_time=300,  # 5 minutes maximum
            init_delay=2.0,
            exp_factor=2.5,
            jitter=True
        )
    )

Adding Status-Specific Configuration
-------------------------------------

To add retry configuration for specific HTTP status codes without affecting existing settings, use the
:py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.add` method.

.. code-block:: python

    # Add 408 (Request Timeout) as a retryable status code
    RetryConfig.add(retryable_status_codes={408})

    # Alternatively add full configuration for 408
    RetryConfig.add(
        status_configs={
            408: RetrySettings(max_attempts=5, init_delay=2.0)
        }
    )

Removing Status Codes From Retry
---------------------------------

To stop retrying specific HTTP status codes, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.remove` method.

.. code-block:: python

    # Stop retrying 500 errors
    RetryConfig.remove(retryable_status_codes={500})

    # Remove specific configuration (falls back to default)
    RetryConfig.remove(status_configs={429})

    # Remove both configuration and retryability
    RetryConfig.remove(
        status_configs={429},
        retryable_status_codes={429}
    )

Replacing All Configuration
----------------------------

To completely replace the retry configuration, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.set` method.

.. warning::
    The :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.set` method **replaces** all existing configuration.
    Use :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.add` to add to existing configuration without removing others.

.. code-block:: python

    # Replace ALL retryable status codes
    RetryConfig.set(retryable_status_codes={429, 500, 503})

    # Replace ALL status-specific configurations
    RetryConfig.set(
        status_configs={
            429: RetrySettings(max_attempts=20),
            500: RetrySettings(max_attempts=5)
        }
    )

Temporary Retry Overrides
~~~~~~~~~~~~~~~~~~~~~~~~~~

For specific operations, you can temporarily override retry behavior using context managers. These overrides only affect
the code within the ``with`` block and don't change the global configuration.

.. note::
    The :py:class:`ibm_watsonx_data_integration.common.retry.RetrySettings` parameter is optional. If not provided, the default retry settings
    will be used for all retryable status codes within the context.

Retrying With Default Settings
-------------------------------

To retry with default settings for all 4xx and 5xx errors, simply use the context manager without specifying custom settings.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import retry_on_http_error

    # Use default retry settings for all errors
    with retry_on_http_error():
        flows = project.flows.get(name='Test Flow')
        project.delete_flow(flow=flow)

Retrying With Custom Settings
------------------------------

To use custom retry settings, provide a :py:class:`ibm_watsonx_data_integration.common.retry.RetrySettings` instance as the first parameter.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import retry_on_http_error, RetrySettings

    # Use custom retry settings for all errors
    with retry_on_http_error(settings=RetrySettings(max_attempts=10, init_delay=2.0)):
        job = project.jobs.get(job_id='some-job-id')
        project.delete_job(job)

Retrying Only Specific Status Codes
------------------------------------

To retry only certain status codes for a specific operation, use the :py:func:`ibm_watsonx_data_integration.common.retry.retry_on_http_error`
context manager with the ``only_status_codes`` parameter.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import retry_on_http_error, RetrySettings

    # Retry ONLY rate limiting errors with custom settings
    with retry_on_http_error(
        settings=RetrySettings(max_attempts=10, init_delay=2.0),
        only_status_codes={429}
    ):
        job = project.jobs.get(job_id='some-job-id')
        project.delete_job(job)

Retrying All Except Specific Status Codes
------------------------------------------

To retry all errors except certain status codes, use the ``skip_status_codes`` parameter.

.. code-block:: python

    # Retry everything except 400 errors
    with retry_on_http_error(
        settings=RetrySettings(max_attempts=5, init_delay=1.0),
        skip_status_codes={400}
    ):
        flows = project.flows
        for flow in flows:
            project.delete_flow(flow=flow)

Retrying All Error Codes
-------------------------

To retry all 4xx and 5xx errors for a specific operation:

.. code-block:: python

    # Retry ALL 4xx and 5xx status codes with custom settings
    with retry_on_http_error(settings=RetrySettings(max_attempts=20, exp_factor=1.5)):
        flows = project.flows.get(name='Test Flow 1')
        project.delete_flow(flow=flow)

    # Retry ALL 4xx and 5xx status codes with default settings
    with retry_on_http_error():
        flows = project.flows.get(name='Test Flow 2')
        project.delete_flow(flow=flow)

Disabling Retry Temporarily
----------------------------

To disable retry for a specific operation, use the :py:func:`ibm_watsonx_data_integration.common.retry.no_retry_on_http_error` context manager.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import no_retry_on_http_error

    # Disable retry for this operation
    with no_retry_on_http_error():
        group_to_delete = platform.access_groups.get(name='Test Access Group')
        pltaform.delete_access_group(access_group=group_to_delete)   # If HTTPError occurs this will fail immediately without retry.

Disabling And Enabling Retries Globally
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Disabling All Retries
---------------------

To disable all HTTP retries globally, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.disable` method.
This saves the current configuration so it can be restored later.

.. code-block:: python

    # Disable all retries
    RetryConfig.disable()

    # Verify retries are disabled
    RetryConfig.show()

Re-enabling Retries
-------------------

To re-enable retries, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.enable` method.

.. code-block:: python

    # Restore to default configuration, this is equivalent to RetryConfig.reset()
    RetryConfig.enable(use_defaults=True)

    # Restore to previous configuration (before disable was called)
    RetryConfig.enable()

Resetting To Factory Defaults
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To reset all retry configuration to factory defaults, use the :py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.reset` method.

.. code-block:: python

    # Reset to factory defaults
    RetryConfig.reset()

    # Verify reset
    RetryConfig.show()


Getting Current Configuration
------------------------------

To retrieve the current configuration as a dictionary, use the
:py:meth:`ibm_watsonx_data_integration.common.retry.RetryConfig.get_current_config` method.

.. code-block:: python

    config = RetryConfig.get_current_config()
    print(config['default'].max_attempts)
    # Output: 3

    print(config['retryable_status_codes'])
    # Output: {429, 500, 502, 503, 504}

    print(config['status_configs'].keys())
    # Output: dict_keys([429])

Understanding Retry Limits
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The retry mechanism enforces the following limits to prevent misuse:

- **Max Attempts:** 1-50
- **Max Time:** 0-3600 seconds (1 hour)
- **Initial Delay:** 0-300 seconds (5 minutes)
- **Exponential Factor:** 1.0-10.0
- **Maximum Delay Between Retries:** 60 seconds

Attempting to configure values outside these limits will raise a :py:class:`ValueError`.

.. code-block:: python

    # This will raise ValueError
    RetryConfig.set(
        default=RetrySettings(max_attempts=100)  # Exceeds MAX_ATTEMPTS_LIMIT of 50
    )

Handling Retry Exceptions
~~~~~~~~~~~~~~~~~~~~~~~~~~

When retries are exhausted, an :py:class:`ibm_watsonx_data_integration.common.retry.HTTPRetryError` exception is raised.
This exception contains information about the retry attempts.

.. code-block:: python

    from ibm_watsonx_data_integration.common.retry import HTTPRetryError

    try:
        flow = project.flows.get(flow_id='some-flow-id')
        project.delete_flow(flow=flow)
    except HTTPRetryError as e:
        print(f"Operation failed after {e.attempts_made} attempts")
        print(f"Total time spent: {e.total_time:.2f} seconds")
        print(f"Last exception: {e.last_exception}")
        if e.last_response:
            print(f"Last response status: {e.last_response.status_code}")

Thread Safety
~~~~~~~~~~~~~

The retry configuration is thread-safe and can be used in multi-threaded applications. Each thread can have its own
temporary override using context managers without affecting other threads.

.. code-block:: python

    import threading

    def worker_thread():
        # This override only affects this thread
        with retry_on_http_error(RetrySettings(max_attempts=5)):
            pipelines = sch.pipelines
            # Process pipelines...

    # Global config remains unchanged
    RetryConfig.set(default=RetrySettings(max_attempts=3))

    # Each thread gets its own override
    threads = [threading.Thread(target=worker_thread) for _ in range(5)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
