# Streaming Flow Sample Scripts

## 1. Authentication and Setup

Before interacting with the SDK, users must authenticate against the IBM Cloud platform. We are showing an example using the IBM Cloud API Key here but other options such as Bearer Token, Zen API Key, and ICP4D Authentication methods are available here.
```python
# Set API Key from IBM Cloud
api_key = 'API KEY'

from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
from ibm_watsonx_data_integration import Platform

auth = IAMAuthenticator(api_key=api_key)
platform = Platform(auth, base_api_url="https://api.ca-tor.dai.cloud.ibm.com")
```

**Explanation:**

- **IAMAuthenticator**: This class handles the security handshake using the user's IBM Cloud API Key.
- **Platform**: This is the entry point for the SDK. It requires the authenticator and the specific base API URL for the instance (e.g., ca-tor for Toronto).
- **Imports**: The script imports datastage services to ensure all flow, stage, and schema objects are available.

## 2. Creating a Project

All flows reside within a Project. This step instantiates a new project where the data flows will be stored.
```python
# Creating a new project
project = platform.create_project(
    name='My first project',
    description='Building sample batch flows',
    tags=['flow_test_project'],
    public=True,
    project_type='wx'
)

project
```

**Explanation:**

- **create_project()**: Creates a new project container. The project_type='wx' parameter specifies this is a watsonx project.
- **Metadata**: Users can attach descriptions and tags immediately to keep their workspace organized.

## 3a. Creating an Environment

Creating a new streaming environment using the latest engine version. We will need an engine to execute a job for the following flow
```python
# Creating a new streaming environment using the latest engine version

environment = project.create_environment(
    name="My first environment", 
    engine_version=platform.available_engine_versions[0], 
    cpus_to_allocate=2, 
    stage_libs=[
        "streamsets-datacollector-basic-lib",
        "streamsets-datacollector-dataformats-lib",
        "streamsets-datacollector-dev-lib",
        # "streamsets-datacollector-aws-lib",
    ]
)
```

**Explanation:**

- **project.create_environment()**: Defines a new virtual execution context (an Environment) where your streaming flows will run.
- **engine_version**: Specifies the exact version of the Data Collector engine to be used.
- **cpus_to_allocate**: Configures the resource allocation for the Engine instance (in this case, requesting 2 CPU cores).
- **stage_libs**: Lists the required libraries, which contain the connectors and processors (stages) used to build the data flow logic.

## 3b. Installing an engine

Generate an install script and run the installation command on any docker/podman or VM of your choosing to deploy an engine. This step is required for a streaming job to run successfully.
```python
# Generate install script and replaces SSET_API_KEY with the API key used above.

environment.get_installation_command(pretty=False).replace("${SSET_API_KEY:?Please provide your API key from IBM Cloud}", f"{auth.api_key}")
```

**Explanation:**

- **get_installation_command()**: Retrieves the complete, ready-to-execute command (e.g., a docker run command) needed to launch the physical Engine instance on a host machine.
- **.replace(...)**: Injects the actual API key (auth.api_key) into the command template, replacing the required environment variable placeholder (${SSET_API_KEY...}) for secure authentication and deployment.

## 4. Creating a Simple Streaming Flow (Dev Raw Data Source → Field Remover → Trash)

This section demonstrates the core workflow: creating a streaming flow object, adding stages, defining simple configurations, and connecting them.
```python
# Create a new Streaming Flow named "Dev to Trash"
# SET environment = environment above if you want to run a job for this flow
flow = project.create_flow(name="Dev to Trash", environment=None, description="")

# Add the Origin (data source) stage
dev_raw_data_source_1 = flow.add_stage("Dev Raw Data Source", type="origin")

# Add the Processor (data transformation) stage
field_remover_1 = flow.add_stage("Field Remover", type="processor")

# Configure the Processor to remove the field at index 1
field_remover_1.fields = ['[1]']

# Add the Destination (data sink) stage
trash_1 = flow.add_stage("Trash", type="destination")

# Connect the Origin output to the Processor input
dev_raw_data_source_1.connect_output_to(field_remover_1)

# Connect the Processor output to the Destination input
field_remover_1.connect_output_to(trash_1)

# Save the complete flow configuration to the project
project.update_flow(flow)
```

**Explanation:**

The code creates and configures a linear Streaming Flow pipeline named "Dev to Trash" using three fundamental stages: an Origin, a Processor, and a Destination.

- **Initialization**: The project.create_flow() method starts by instantiating the pipeline object.
- **Stages**: Three stages are added to the flow:
  - **Dev Raw Data Source** (type origin): Acts as the data input source (the start of the stream).
  - **Field Remover** (type processor): Configured with field_remover_1.fields = ['[1]'], this stage actively transforms the data stream by removing the field at index 1.
  - **Trash** (type destination): Acts as the data output sink, where the final, processed records are discarded.
- **Connections**: The connect_output_to() calls establish the data flow: Origin → Processor → Destination.
- **Saving**: Finally, project.update_flow(flow) saves the newly configured pipeline structure and its settings to the project, making it ready for execution.

## 5. Running a job for the basic flow

Note: The job for the basic flow will only run if you have successfully created an environment and installed an engine. Refer to the documentation here for additional help
```python
# Create a Job object associated with the existing 'flow'.
dev_to_trash_job = project.create_job(
    name="DevToTrash_StreamingJob",  
    flow=streaming_flow                 
)

# Start a Job Run (an instance of execution) for the created Job.
dev_to_trash_job_run = dev_to_trash_job.start(
    name="DevToTrash_JobRun_01",
    description="Initial test run for the basic Dev to Trash pipeline"
)

print(f"Streaming Job '{dev_to_trash_job.name}' started with Run ID: {dev_to_trash_job_run.job_run_id}")
```

## 6. Code Generation for Streaming Flows (Reverse Engineering)

This powerful feature allows users to take an existing flow and generate the Python SDK code required to rebuild it.
```python
# Python generator generating streaming flow template or learning SDK
generator = PythonGenerator(
    source=streaming_flow,
    destination='../Desktop/streaming_flow_template.py',
    auth=auth,  
    base_api_url='https://api.ca-tor.dai.cloud.ibm.com'
)
generator.save()
```

**Explanation:**

- **PythonGenerator**: Useful for migration or learning. A user can design a flow visually in the UI, export it, and use this tool to see how to write the equivalent SDK code.

## 7. Complex Streaming Flow Example (Source -> Transform -> Multiple Targets)

This code defines a Streaming Flow for real-time fraud detection.
```python
import os
from ibm_watsonx_data_integration import Platform
from ibm_watsonx_data_integration.common.auth import IAMAuthenticator

# Retrieve an existing project by its unique ID. All assets, including flows, reside in a project.
project = platform.projects.get(project_id="be4a2f4f-6007-4fe0-b99e-46ea67b12c84")

# Create a new Streaming Flow container for the fraud detection pipeline.
flow = project.create_flow(name="Streaming data for fraud detection flow rebuild", environment=None, description="")

# --- Pipeline Stages Configuration ---

# 1. Origin: Kafka Multitopic Consumer
kafka_multitopic_consumer_1 = flow.add_stage("Kafka Multitopic Consumer")
kafka_multitopic_consumer_1.max_batch_size_in_records = 1  # Configure for real-time, record-by-record processing
kafka_multitopic_consumer_1.broker_uri = "kafka:9092"      # Specifies the Kafka broker location
kafka_multitopic_consumer_1.topic_list = ['new_credit_card_transactions']  # Subscribes to the transaction topic
kafka_multitopic_consumer_1.data_format = 'JSON'           # Expects incoming data to be in JSON format
# Explanation: This stage extracts new credit card transactions in real-time from a Kafka queue.

# 2. Processor: JDBC Lookup (SingleStore)
jdbc_lookup_1 = flow.add_stage("JDBC Lookup")
jdbc_lookup_1.jdbc_connection_string = "jdbc:singlestore://10.89.0.2:3306/finance"  # Connection to SingleStore DB
jdbc_lookup_1.sql_query = """WITH prev AS (
    SELECT location, timestamp  
    FROM transactions 
    WHERE account_id = ${record:value('/account_id')} AND suspicious = FALSE
    ORDER BY transaction_id DESC 
    LIMIT 1
)
SELECT  
    GEOGRAPHY_DISTANCE(prev.location, "${record:value('/location')}") as delta_x,  
    TIMESTAMPDIFF(SECOND, prev.timestamp, "${record:value('/timestamp')}") as delta_t 
FROM prev;"""
# Explanation: This stage looks up the *last non-suspicious transaction* for the current account ID, calculates the geographical distance (delta_x) and time difference (delta_t), and appends these values to the current record.

# 3. Processor: Expression Evaluator
expression_evaluator_1 = flow.add_stage("Expression Evaluator")
expression_evaluator_1.field_expressions = [{'fieldToSet': '/suspicious', 'expression': "${record:exists('/delta_x') and record:exists('/delta_t') and \n  record:value('/delta_x') / record:value('/delta_t') > 268.224}"}]
# Explanation: This is the core fraud logic. It creates a new field '/suspicious' and sets it to TRUE if the calculated velocity (delta_x / delta_t) exceeds a threshold (268.224 m/s, or ~600 mph).

# 4. Destination (Split A): Elasticsearch
elasticsearch_1 = flow.add_stage("Elasticsearch")
elasticsearch_1.index = "transactions"
elasticsearch_1.http_urls = "http://10.89.0.196:9200"
# Explanation: This stage sends the processed, labeled transaction to Elasticsearch, likely for near real-time visualization and analysis by fraud teams.

# 5. Destination (Split B): SingleStore
singlestore_1 = flow.add_stage("SingleStore")
singlestore_1.schema_name = "finance"
singlestore_1.table_name = "transactions"
singlestore_1.jdbc_connection_string = "jdbc:singlestore://10.89.0.2:3306/finance"
# Explanation: This stage writes the fully processed and labeled transaction back into the main transactional database for persistence and future lookups (used by the JDBC Lookup stage).

# --- Data Flow Connections ---

# Connect Kafka (Origin) to JDBC Lookup (Processor)
kafka_multitopic_consumer_1.connect_output_to(jdbc_lookup_1)

# Connect JDBC Lookup (Processor) to Expression Evaluator (Processor)
jdbc_lookup_1.connect_output_to(expression_evaluator_1)

# Split the output from the Expression Evaluator to two Destinations simultaneously
expression_evaluator_1.connect_output_to(elasticsearch_1)
expression_evaluator_1.connect_output_to(singlestore_1)

# Save all stage definitions and connections to the project
project.update_flow(flow)
```

**Explanation:**

This Streaming Flow implements a real-time system to detect credit card fraud based on transaction velocity. The pipeline follows an Extract → Enrich → Transform → Load (Split) pattern:

- **Extract (Kafka Consumer)**: New transaction records are extracted in real-time from the new_credit_card_transactions Kafka topic.
- **Enrichment (JDBC Lookup)**: For each new transaction, the JDBC Lookup stage queries the SingleStore database to find the location and timestamp of the account's last non-suspicious transaction. It then calculates two critical metrics:
- **Transformation (Expression Evaluator)**: This stage applies the fraud detection rule. It calculates the transaction velocity and marks the transaction as suspicious = TRUE if the calculated speed exceeds the predefined threshold. This is known as a velocity check.
- **Load (Split Destinations)**: The processed, fraud-labeled transaction record is then simultaneously written to two different systems:
  - **Elasticsearch**: For immediate visualization and monitoring by fraud analysts.
  - **SingleStore**: To persist the new transaction (including its suspicious flag), making it available for future lookups by the JDBC Lookup stage.

This flow is a great example of using streaming data, transactional lookups, and real-time processing to make a critical business decision (fraud detection) on a per-record basis.