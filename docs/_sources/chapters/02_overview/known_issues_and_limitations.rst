.. _overview__known_issues_and_limitations:

Known Issues and limitations
============================

A list of currently known issues and limitations can be found below.
This page is actively maintained and updated with each release of the ``ibm-watsonx-data-integration`` SDK for Python.

Known Issues
~~~~~~~~~~~~

* Users may encounter HTTP 429 errors ('too many requests') when making calls from the SDK - this is actively being mitigated and will be fixed in the next release.
* The Streaming Python Generator must use triple quotes (""") for SQL strings to avoid syntax issues.
* When running the Streaming Python Generator on a flow using a JSON data format, it currently returns a JSON literal instead of a "JSON" string.

Limitations
~~~~~~~~~~~

* The SDK only supports the IAM API key authentication mechanism at present.
* Currently, the SDK only supports creating a flow with an engine installed.
