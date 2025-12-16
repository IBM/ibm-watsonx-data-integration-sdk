# Batch Flow Sample Script

## 1. Authentication and Setup

Before interacting with the SDK, users must authenticate against the IBM Cloud platform. We are showing an example using the IBM Cloud API Key here but other options such as Bearer Token, Zen API Key, and ICP4D Authentication methods are available here.

```python
# Set API Key from IBM Cloud
api_key = 'API KEY'

from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
from ibm_watsonx_data_integration import Platform
from ibm_watsonx_data_integration.services.datastage import *

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
# Creating 
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

## 3. Creating a Simple Batch Flow (Row Gen -> Peek)

This section demonstrates the core workflow: creating a flow object, adding stages, connecting them via links, and defining the data schema.

```python
# Flow
flow = project.create_flow(
    name="RowGenPeek",
    environment=None,
    flow_type="batch"
)

# Stages
row_generator = flow.add_stage("Row Generator", "Row_Generator")
row_generator.configuration.records = 5
row_generator.configuration.runtime_column_propagation = False

peek = flow.add_stage("Peek", "Peek")
peek.configuration.runtime_column_propagation = False

# Graph
link_1 = row_generator.connect_output_to(peek)
link_1.name = "Link_1"

row_generator_schema = link_1.create_schema()
row_generator_schema.add_field("VARCHAR", "COLUMN_1").length(100)

project.update_flow(flow)
```

**Explanation:**

- **create_flow**: Initializes the flow in memory. Note that flow_type="batch" is strictly required.
- **add_stage**: Adds specific processing nodes. The first argument is the official Stage Type (e.g., "Row Generator"), and the second is the user-defined Label.
- **runtime_column_propagation = False**: This is a critical setting. It disables the automatic "guessing" of columns, requiring the user to explicitly define the schema. This provides greater control over the pipeline.
- **connect_output_to**: Establishes the directional link between the source (row_generator) and the target (peek).
- **create_schema**: A schema belongs to the link between stages, not the stages themselves. Here, we define that Link_1 carries a single VARCHAR column.

## 4. Modifying and Persisting Configurations

The SDK allows for in-memory modification of flow properties, but these changes must be pushed to the server to take effect.

```python
# Access the stage configuration
row_generator.configuration.records = 10

# Persist the change
project.update_flow(flow)
```

**Explanation:**

- **In-Memory Editing**: You can directly access properties like .configuration.records on the stage object.
- **update_flow(flow)**: This is the crucial commit step. Until this is called, changes like increasing the record count to 10 exist only in the local Python session. This method saves the updated flow definition to the backend.

## 5. Running a Job

To execute a flow, it must be run as a "Job".

```python
row_gen_peek_job = project.create_job(name="RowGenPeek_job", flow=flow)
job_start = row_gen_peek_job.start(name="RowGenPeek_job_run", description="")

print(f"Job name: {job_start.job_name}") 
print(f"Job state: {job_start.state}")
```

**Explanation:**

- **Auto-Compilation**: Creating a job out of a flow automatically compiles it. You do not need to call compile() manually before this step.
- **create_job & start**: This encapsulates the execution logic. The job_start object returns immediate feedback on the state (e.g., "running", "completed").

## 6. Code Generation for Batch Flows (Reverse Engineering)

This powerful feature allows users to take an existing flow (exported as a zip) and generate the Python SDK code required to rebuild it.

```python
from ibm_watsonx_data_integration.services.datastage.codegen import PythonGenerator

code_gen = PythonGenerator()
code_gen.configuration.mode = "file_per_flow"
code_gen.generate(input_path="RowGenPeek.zip", output_path="generated_code")
```

**Explanation:**

- **PythonGenerator**: Useful for migration or learning. A user can design a flow visually in the UI, export it, and use this tool to see how to write the equivalent SDK code.

## 7. Complex Flow Example (Source -> Transform -> Target)

This section of the script builds a realistic ETL pipeline: loading transactions, filtering high-value items, transforming them, and saving to multiple targets (MySQL and a File). Note: A job with this flow will not run as the source credentials are dummy credentials.

**Key Flow Components:**

- **Source (PostgreSQL)**: Configured with connection details (hostname, port, credentials). Note the use of connection.defer_credentials = False to embed credentials for the run.
- **Filter Stage**: Splits data based on logic.
  - where_properties = [{"where": "amount > 750"}]: Filters for high-value transactions.
- **Transformer**: Used for data mapping or modification.
- **Targets**:
  - **MySQL**: A database target for processed rows.
  - **Sequential File**: A file system target for logging or archiving.

**Schema Mapping**: The script demonstrates explicit schema mapping using .source():

This .source("Link_X.field") syntax tells the SDK exactly which upstream column populates the downstream field, ensuring data lineage is maintained across the Transformer and Filter stages.

```python
# Periodically load all transactions from PostgreSQL (historical and recent).
# Train fraud detection models (e.g., logistic regression, random forest) 
project = platform.projects.get(name='My first project')

flow = project.create_flow(name="TransactionsHighValue-SDK", environment=None, flow_type="batch")

# Stages
transactions_1 = flow.add_stage("PostgreSQL", "transactions_1")
transactions_1.configuration.runtime_column_propagation = False
transactions_1.configuration.table_name = "transactions"
transactions_1.configuration.connection.name = "PostgreSQL_test"
transactions_1.configuration.connection.database = "xxx"
transactions_1.configuration.connection.defer_credentials = False
transactions_1.configuration.connection.hostname_or_ip_address = "00.000.000.001"
transactions_1.configuration.connection.password = "Password"
transactions_1.configuration.connection.port = "8080"
transactions_1.configuration.connection.proxy = False
transactions_1.configuration.connection.port_is_ssl_enabled = False
transactions_1.configuration.connection.username = "cpd"

peek_1 = flow.add_stage("Peek", "Peek_1")

peek_1.configuration.runtime_column_propagation = False
peek_1.configuration.outputlink_ordering_list = [{"link_label": "Output 1", "link_name": "Link_2"}]

filter_1 = flow.add_stage("Filter", "Filter_1")

filter_1.configuration.runtime_column_propagation = False
filter_1.configuration.show_coll_type = False
filter_1.configuration.show_part_type = True
filter_1.configuration.show_sort_options = False
filter_1.configuration.where_properties = [{"where": "amount > 750", "target": "0"}]

transformer_1 = flow.add_stage("Transformer", "Transformer_1")
transformer_1.configuration.runtime_column_propagation = False

tm_ds_db_1_1 = flow.add_stage("MySQL", "TM_DS_DB_1_1")
tm_ds_db_1_1.configuration.runtime_column_propagation = False
tm_ds_db_1_1.configuration.column_metadata_change_propagation = False
tm_ds_db_1_1.configuration.defer_credentials = "false"
tm_ds_db_1_1.configuration.output_acp_should_hide = False
tm_ds_db_1_1.configuration.schema_name = "TM_DS_DB_1"
tm_ds_db_1_1.configuration.show_coll_type = False
tm_ds_db_1_1.configuration.show_part_type = True
tm_ds_db_1_1.configuration.show_sort_options = False
tm_ds_db_1_1.configuration.table_name = "processed_transactions"

tm_ds_db_1_1.configuration.connection.name = "MySQL Legacy Financial DB"
tm_ds_db_1_1.configuration.connection.database = "TM_DS_DB_1"
tm_ds_db_1_1.configuration.connection.defer_credentials = "false"
tm_ds_db_1_1.configuration.connection.hostname_or_ip_address = "00.000.000.002"
tm_ds_db_1_1.configuration.connection.password = "Password2"
tm_ds_db_1_1.configuration.connection.port = "80801"
tm_ds_db_1_1.configuration.connection.proxy = False
tm_ds_db_1_1.configuration.connection.port_is_ssl_enabled = True
tm_ds_db_1_1.configuration.connection.username = "TM_DS_USER"

peek_2 = flow.add_stage("Peek", "Peek_2")

peek_2.configuration.runtime_column_propagation = False
peek_2.configuration.outputlink_ordering_list = [{"link_label": "Output 1", "link_name": "Link_6"}]

sequential_file_1 = flow.add_stage("Sequential file", "Sequential_file_1")

sequential_file_1.configuration.runtime_column_propagation = False
sequential_file_1.configuration.file = ["/ds-storage/high_value_transactions_103030.csv"]
sequential_file_1.configuration.first_line_is_column_names = SEQUENTIALFILE.FirstLineColumnNames.true
sequential_file_1.configuration.null_field_value = "'NULL'"
sequential_file_1.configuration.show_coll_type = True
sequential_file_1.configuration.show_part_type = False
sequential_file_1.configuration.show_sort_options = True

# Graph
link_1 = transactions_1.connect_output_to(peek_1)
link_1.name = "Link_1"
transactions_1_schema = link_1.create_schema()
transactions_1_schema.add_field("INTEGER", "id")
transactions_1_schema.add_field("INTEGER", "account_id")
transactions_1_schema.add_field("VARCHAR", "timestamp").length(50)
transactions_1_schema.add_field("NUMERIC", "amount").length(10).scale(2)
transactions_1_schema.add_field("LONGVARCHAR", "location").length(1024)


link_2 = peek_1.connect_output_to(filter_1)
link_2.name = "Link_2"
peek_1_schema = link_2.create_schema()
peek_1_schema.add_field("INTEGER", "id").source("Link_1.id")
peek_1_schema.add_field("INTEGER", "account_id").source("Link_1.account_id")
peek_1_schema.add_field("VARCHAR", "timestamp").source("Link_1.timestamp").length(50)
peek_1_schema.add_field("NUMERIC", "amount").source("Link_1.amount").length(10).scale(2)
peek_1_schema.add_field("LONGVARCHAR", "location").source("Link_1.location").length(1024)


link_3 = filter_1.connect_output_to(transformer_1)
link_3.name = "Link_3"
filter_1_schema = link_3.create_schema()
filter_1_schema.add_field("INTEGER", "id").source("Link_2.id")
filter_1_schema.add_field("INTEGER", "account_id").source("Link_2.account_id")
filter_1_schema.add_field("VARCHAR", "timestamp").source("Link_2.timestamp").length(50)
filter_1_schema.add_field("NUMERIC", "amount").source("Link_2.amount").length(10).scale(2)
filter_1_schema.add_field("LONGVARCHAR", "location").source("Link_2.location").length(1024)


link_4 = transformer_1.connect_output_to(tm_ds_db_1_1)
link_4.name = "Link_4"
transformer_1_schema = link_4.create_schema()
transformer_1_schema.add_field("INTEGER", "account_id").source("Link_3.account_id")
transformer_1_schema.add_field("VARCHAR", "timestamp").source("Link_3.timestamp").length(50)
transformer_1_schema.add_field("NUMERIC", "amount").source("Link_3.amount").length(10).scale(2)
transformer_1_schema.add_field("LONGVARCHAR", "location").source("Link_3.location").length(1024)

link_5 = transformer_1.connect_output_to(peek_2)
link_5.name = "Link_5"
transformer_1_schema_2 = link_5.create_schema()
transformer_1_schema_2.add_field("INTEGER", "id").source("Link_3.id")
transformer_1_schema_2.add_field("INTEGER", "account_id").source("Link_3.account_id")
transformer_1_schema_2.add_field("VARCHAR", "timestamp").source("Link_3.timestamp").length(50)
transformer_1_schema_2.add_field("NUMERIC", "amount").source("Link_3.amount").length(10).scale(2)
transformer_1_schema_2.add_field("LONGVARCHAR", "location").source("Link_3.location").length(1024)


link_6 = peek_2.connect_output_to(sequential_file_1)
link_6.name = "Link_6"
peek_2_schema = link_6.create_schema()
peek_2_schema.add_field("INTEGER", "id").source("Link_5.id")
peek_2_schema.add_field("INTEGER", "account_id").source("Link_5.account_id")
peek_2_schema.add_field("VARCHAR", "timestamp").source("Link_5.timestamp").length(50)
peek_2_schema.add_field("NUMERIC", "amount").source("Link_5.amount").length(10).scale(2)
peek_2_schema.add_field("LONGVARCHAR", "location").source("Link_5.location").length(1024)


project.update_flow(flow)
print(f"New '{flow.name}' has been created")
```

## 8. End-to-End Execution

The final section of the script puts it all together in a single execution block:

1. Authenticate.
2. Create Project.
3. Define Flow & Stages (RowGen + Peek).
4. Connect & Define Schema.
5. Persist (update_flow).
6. Run (create_job -> start).

```python
# Authentication
from ibm_watsonx_data_integration.common.auth import IAMAuthenticator
from ibm_watsonx_data_integration import Platform
from ibm_watsonx_data_integration.services.datastage import *

auth = IAMAuthenticator(api_key=api_key)
platform = Platform(auth, base_api_url="https://api.ca-tor.dai.cloud.ibm.com")

# Create Project
project = platform.create_project(
    name="My first project",
    description="Building sample batch flows",
    tags=["flow_test_project"],
    public=True,
    project_type="wx"
)

print(project)  # Expected: Project(guid='xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', name='My first project')

# Create Flow
flow = project.create_flow(
    name="RowGenPeek",
    environment=None,
    flow_type="batch"
)

# Stages
row_generator = flow.add_stage("Row Generator", "Row_Generator")
row_generator.configuration.runtime_column_propagation = False

peek = flow.add_stage("Peek", "Peek")
peek.configuration.runtime_column_propagation = False

# Graph and Schema
link_1 = row_generator.connect_output_to(peek)
link_1.name = "Link_1"

row_generator_schema = link_1.create_schema()
row_generator_schema.add_field("VARCHAR", "COLUMN_1").length(100)

# Persist flow
project.update_flow(flow)
print("Flow created successfully:", flow.name)


# Create and Run Job
row_gen_peek_job = project.create_job(name="RowGenPeek_job", flow=flow)
job_start = row_gen_peek_job.start(
    name="RowGenPeek_job_run",
    description=""
)

print("Job name:", job_start.job_name)
print("Job state:", job_start.state)
```