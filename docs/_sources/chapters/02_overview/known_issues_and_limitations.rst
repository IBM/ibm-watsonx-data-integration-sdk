.. _overview__known_issues_and_limitations:

Known Issues and limitations
============================

A list of currently known issues and limitations can be found below.
This page is actively maintained and updated with each release of the ``ibm-watsonx-data-integration`` SDK for Python.

Known Issues
~~~~~~~~~~~~

* Users may encounter HTTP 429 errors ('too many requests') when making calls from the SDK - this is actively being mitigated and will be fixed in the next release.

Limitations
~~~~~~~~~~~

* The SDK only supports the IAM API key authentication mechanism at present.
* The SDK only fully supports StreamSets (real-time streaming) functionality at this time.
  Support for additional services and components, like DataStage (batch processing) is planned for future releases.
* Currently, the SDK only supports creating a flow with an engine installed.
