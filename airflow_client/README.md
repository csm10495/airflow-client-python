<!--
 Licensed to the Apache Software Foundation (ASF) under one
 or more contributor license agreements.  See the NOTICE file
 distributed with this work for additional information
 regarding copyright ownership.  The ASF licenses this file
 to you under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance
 with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing,
 software distributed under the License is distributed on an
 "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 KIND, either express or implied.  See the License for the
 specific language governing permissions and limitations
 under the License.
 -->

# Apache Airflow Python Client
# Overview

To facilitate management, Apache Airflow supports a range of REST API endpoints across its
objects.
This section provides an overview of the API design, methods, and supported use cases.

Most of the endpoints accept `JSON` as input and return `JSON` responses.
This means that you must usually add the following headers to your request:
```
Content-type: application/json
Accept: application/json
```

## Resources

The term `resource` refers to a single type of object in the Airflow metadata. An API is broken up by its
endpoint's corresponding resource.
The name of a resource is typically plural and expressed in camelCase. Example: `dagRuns`.

Resource names are used as part of endpoint URLs, as well as in API parameters and responses.

## CRUD Operations

The platform supports **C**reate, **R**ead, **U**pdate, and **D**elete operations on most resources.
You can review the standards for these operations and their standard parameters below.

Some endpoints have special behavior as exceptions.

### Create

To create a resource, you typically submit an HTTP `POST` request with the resource's required metadata
in the request body.
The response returns a `201 Created` response code upon success with the resource's metadata, including
its internal `id`, in the response body.

### Read

The HTTP `GET` request can be used to read a resource or to list a number of resources.

A resource's `id` can be submitted in the request parameters to read a specific resource.
The response usually returns a `200 OK` response code upon success, with the resource's metadata in
the response body.

If a `GET` request does not include a specific resource `id`, it is treated as a list request.
The response usually returns a `200 OK` response code upon success, with an object containing a list
of resources' metadata in the response body.

When reading resources, some common query parameters are usually available. e.g.:
```
v1/connections?limit=25&offset=25
```

|Query Parameter|Type|Description|
|---------------|----|-----------|
|limit|integer|Maximum number of objects to fetch. Usually 25 by default|
|offset|integer|Offset after which to start returning objects. For use with limit query parameter.|

### Update

Updating a resource requires the resource `id`, and is typically done using an HTTP `PATCH` request,
with the fields to modify in the request body.
The response usually returns a `200 OK` response code upon success, with information about the modified
resource in the response body.

### Delete

Deleting a resource requires the resource `id` and is typically executing via an HTTP `DELETE` request.
The response usually returns a `204 No Content` response code upon success.

## Conventions

- Resource names are plural and expressed in camelCase.
- Names are consistent between URL parameter name and field name.

- Field names are in snake_case.
```json
{
    \"name\": \"string\",
    \"slots\": 0,
    \"occupied_slots\": 0,
    \"used_slots\": 0,
    \"queued_slots\": 0,
    \"open_slots\": 0
}
```

### Update Mask

Update mask is available as a query parameter in patch endpoints. It is used to notify the
API which fields you want to update. Using `update_mask` makes it easier to update objects
by helping the server know which fields to update in an object instead of updating all fields.
The update request ignores any fields that aren't specified in the field mask, leaving them with
their current values.

Example:
```
  resource = request.get('/resource/my-id').json()
  resource['my_field'] = 'new-value'
  request.patch('/resource/my-id?update_mask=my_field', data=json.dumps(resource))
```

## Versioning and Endpoint Lifecycle

- API versioning is not synchronized to specific releases of the Apache Airflow.
- APIs are designed to be backward compatible.
- Any changes to the API will first go through a deprecation phase.

# Trying the API

You can use a third party client, such as [curl](https://curl.haxx.se/), [HTTPie](https://httpie.org/),
[Postman](https://www.postman.com/) or [the Insomnia rest client](https://insomnia.rest/) to test
the Apache Airflow API.

Note that you will need to pass credentials data.

For e.g., here is how to pause a DAG with [curl](https://curl.haxx.se/), when basic authorization is used:
```bash
curl -X PATCH 'https://example.com/api/v1/dags/{dag_id}?update_mask=is_paused' \\
-H 'Content-Type: application/json' \\
--user \"username:password\" \\
-d '{
    \"is_paused\": true
}'
```

Using a graphical tool such as [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/),
it is possible to import the API specifications directly:

1. Download the API specification by clicking the **Download** button at top of this document
2. Import the JSON specification in the graphical tool of your choice.
  - In *Postman*, you can click the **import** button at the top
  - With *Insomnia*, you can just drag-and-drop the file on the UI

Note that with *Postman*, you can also generate code snippets by selecting a request and clicking on
the **Code** button.

## Enabling CORS

[Cross-origin resource sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
is a browser security feature that restricts HTTP requests that are
initiated from scripts running in the browser.

For details on enabling/configuring CORS, see
[Enabling CORS](https://airflow.apache.org/docs/apache-airflow/stable/security/api.html).

# Authentication

To be able to meet the requirements of many organizations, Airflow supports many authentication methods,
and it is even possible to add your own method.

If you want to check which auth backend is currently set, you can use
`airflow config get-value api auth_backends` command as in the example below.
```bash
$ airflow config get-value api auth_backends
airflow.api.auth.backend.basic_auth
```
The default is to deny all requests.

For details on configuring the authentication, see
[API Authorization](https://airflow.apache.org/docs/apache-airflow/stable/security/api.html).

# Errors

We follow the error response format proposed in [RFC 7807](https://tools.ietf.org/html/rfc7807)
also known as Problem Details for HTTP APIs. As with our normal API responses,
your client must be prepared to gracefully handle additional members of the response.

## Unauthenticated

This indicates that the request has not been applied because it lacks valid authentication
credentials for the target resource. Please check that you have valid credentials.

## PermissionDenied

This response means that the server understood the request but refuses to authorize
it because it lacks sufficient rights to the resource. It happens when you do not have the
necessary permission to execute the action you performed. You need to get the appropriate
permissions in other to resolve this error.

## BadRequest

This response means that the server cannot or will not process the request due to something
that is perceived to be a client error (e.g., malformed request syntax, invalid request message
framing, or deceptive request routing). To resolve this, please ensure that your syntax is correct.

## NotFound

This client error response indicates that the server cannot find the requested resource.

## MethodNotAllowed

Indicates that the request method is known by the server but is not supported by the target resource.

## NotAcceptable

The target resource does not have a current representation that would be acceptable to the user
agent, according to the proactive negotiation header fields received in the request, and the
server is unwilling to supply a default representation.

## AlreadyExists

The request could not be completed due to a conflict with the current state of the target
resource, e.g. the resource it tries to create already exists.

## Unknown

This means that the server encountered an unexpected condition that prevented it from
fulfilling the request.


This Python package is automatically generated by the [OpenAPI Generator](https://openapi-generator.tech) project:

- API version: 1.0.0
- Package version: 2.3.0
- Build package: org.openapitools.codegen.languages.PythonClientCodegen
For more information, please visit [https://airflow.apache.org](https://airflow.apache.org)

## Requirements.

Python >= 3.6

## Installation & Usage
### pip install

If the python package is hosted on a repository, you can install directly using:

```sh
pip install git+https://github.com/apache/airflow-client-python.git
```
(you may need to run `pip` with root permission: `sudo pip install git+https://github.com/apache/airflow-client-python.git`)

Then import the package:
```python
import airflow_client.client
```

### Setuptools

Install via [Setuptools](http://pypi.python.org/pypi/setuptools).

```sh
python setup.py install --user
```
(or `sudo python setup.py install` to install the package for all users)

Then import the package:
```python
import airflow_client.client
```

## Getting Started

Please follow the [installation procedure](#installation--usage) and then run the following:

```python

import time
import airflow_client.client
from pprint import pprint
from airflow_client.client.api import config_api
from airflow_client.client.model.config import Config
from airflow_client.client.model.error import Error
# Defining the host is optional and defaults to http://localhost/api/v1
# See configuration.py for a list of all supported configuration parameters.
configuration = client.Configuration(
    host = "http://localhost/api/v1"
)

# The client must configure the authentication and authorization parameters
# in accordance with the API server security policy.
# Examples for each auth method are provided below, use the example that
# satisfies your auth use case.

# Configure HTTP basic authorization: Basic
configuration = client.Configuration(
    username = 'YOUR_USERNAME',
    password = 'YOUR_PASSWORD'
)


# Enter a context with an instance of the API client
with client.ApiClient(configuration) as api_client:
    # Create an instance of the API class
    api_instance = config_api.ConfigApi(api_client)
    
    try:
        # Get current configuration
        api_response = api_instance.get_config()
        pprint(api_response)
    except client.ApiException as e:
        print("Exception when calling ConfigApi->get_config: %s\n" % e)
```

## Documentation for API Endpoints

All URIs are relative to *http://localhost/api/v1*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*ConfigApi* | [**get_config**](docs/ConfigApi.md#get_config) | **GET** /config | Get current configuration
*ConnectionApi* | [**delete_connection**](docs/ConnectionApi.md#delete_connection) | **DELETE** /connections/{connection_id} | Delete a connection
*ConnectionApi* | [**get_connection**](docs/ConnectionApi.md#get_connection) | **GET** /connections/{connection_id} | Get a connection
*ConnectionApi* | [**get_connections**](docs/ConnectionApi.md#get_connections) | **GET** /connections | List connections
*ConnectionApi* | [**patch_connection**](docs/ConnectionApi.md#patch_connection) | **PATCH** /connections/{connection_id} | Update a connection
*ConnectionApi* | [**post_connection**](docs/ConnectionApi.md#post_connection) | **POST** /connections | Create a connection
*ConnectionApi* | [**test_connection**](docs/ConnectionApi.md#test_connection) | **POST** /connections/test | Test a connection
*DAGApi* | [**delete_dag**](docs/DAGApi.md#delete_dag) | **DELETE** /dags/{dag_id} | Delete a DAG
*DAGApi* | [**get_dag**](docs/DAGApi.md#get_dag) | **GET** /dags/{dag_id} | Get basic information about a DAG
*DAGApi* | [**get_dag_details**](docs/DAGApi.md#get_dag_details) | **GET** /dags/{dag_id}/details | Get a simplified representation of DAG
*DAGApi* | [**get_dag_source**](docs/DAGApi.md#get_dag_source) | **GET** /dagSources/{file_token} | Get a source code
*DAGApi* | [**get_dags**](docs/DAGApi.md#get_dags) | **GET** /dags | List DAGs
*DAGApi* | [**get_task**](docs/DAGApi.md#get_task) | **GET** /dags/{dag_id}/tasks/{task_id} | Get simplified representation of a task
*DAGApi* | [**get_tasks**](docs/DAGApi.md#get_tasks) | **GET** /dags/{dag_id}/tasks | Get tasks for DAG
*DAGApi* | [**patch_dag**](docs/DAGApi.md#patch_dag) | **PATCH** /dags/{dag_id} | Update a DAG
*DAGApi* | [**patch_dags**](docs/DAGApi.md#patch_dags) | **PATCH** /dags | Update DAGs
*DAGApi* | [**post_clear_task_instances**](docs/DAGApi.md#post_clear_task_instances) | **POST** /dags/{dag_id}/clearTaskInstances | Clear a set of task instances
*DAGApi* | [**post_set_task_instances_state**](docs/DAGApi.md#post_set_task_instances_state) | **POST** /dags/{dag_id}/updateTaskInstancesState | Set a state of task instances
*DAGRunApi* | [**delete_dag_run**](docs/DAGRunApi.md#delete_dag_run) | **DELETE** /dags/{dag_id}/dagRuns/{dag_run_id} | Delete a DAG run
*DAGRunApi* | [**get_dag_run**](docs/DAGRunApi.md#get_dag_run) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id} | Get a DAG run
*DAGRunApi* | [**get_dag_runs**](docs/DAGRunApi.md#get_dag_runs) | **GET** /dags/{dag_id}/dagRuns | List DAG runs
*DAGRunApi* | [**get_dag_runs_batch**](docs/DAGRunApi.md#get_dag_runs_batch) | **POST** /dags/~/dagRuns/list | List DAG runs (batch)
*DAGRunApi* | [**post_dag_run**](docs/DAGRunApi.md#post_dag_run) | **POST** /dags/{dag_id}/dagRuns | Trigger a new DAG run
*DAGRunApi* | [**update_dag_run_state**](docs/DAGRunApi.md#update_dag_run_state) | **PATCH** /dags/{dag_id}/dagRuns/{dag_run_id} | Modify a DAG run
*EventLogApi* | [**get_event_log**](docs/EventLogApi.md#get_event_log) | **GET** /eventLogs/{event_log_id} | Get a log entry
*EventLogApi* | [**get_event_logs**](docs/EventLogApi.md#get_event_logs) | **GET** /eventLogs | List log entries
*ImportErrorApi* | [**get_import_error**](docs/ImportErrorApi.md#get_import_error) | **GET** /importErrors/{import_error_id} | Get an import error
*ImportErrorApi* | [**get_import_errors**](docs/ImportErrorApi.md#get_import_errors) | **GET** /importErrors | List import errors
*MonitoringApi* | [**get_health**](docs/MonitoringApi.md#get_health) | **GET** /health | Get instance status
*MonitoringApi* | [**get_version**](docs/MonitoringApi.md#get_version) | **GET** /version | Get version information
*PermissionApi* | [**get_permissions**](docs/PermissionApi.md#get_permissions) | **GET** /permissions | List permissions
*PluginApi* | [**get_plugins**](docs/PluginApi.md#get_plugins) | **GET** /plugins | Get a list of loaded plugins
*PoolApi* | [**delete_pool**](docs/PoolApi.md#delete_pool) | **DELETE** /pools/{pool_name} | Delete a pool
*PoolApi* | [**get_pool**](docs/PoolApi.md#get_pool) | **GET** /pools/{pool_name} | Get a pool
*PoolApi* | [**get_pools**](docs/PoolApi.md#get_pools) | **GET** /pools | List pools
*PoolApi* | [**patch_pool**](docs/PoolApi.md#patch_pool) | **PATCH** /pools/{pool_name} | Update a pool
*PoolApi* | [**post_pool**](docs/PoolApi.md#post_pool) | **POST** /pools | Create a pool
*ProviderApi* | [**get_providers**](docs/ProviderApi.md#get_providers) | **GET** /providers | List providers
*RoleApi* | [**delete_role**](docs/RoleApi.md#delete_role) | **DELETE** /roles/{role_name} | Delete a role
*RoleApi* | [**get_role**](docs/RoleApi.md#get_role) | **GET** /roles/{role_name} | Get a role
*RoleApi* | [**get_roles**](docs/RoleApi.md#get_roles) | **GET** /roles | List roles
*RoleApi* | [**patch_role**](docs/RoleApi.md#patch_role) | **PATCH** /roles/{role_name} | Update a role
*RoleApi* | [**post_role**](docs/RoleApi.md#post_role) | **POST** /roles | Create a role
*TaskInstanceApi* | [**get_extra_links**](docs/TaskInstanceApi.md#get_extra_links) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/links | List extra links
*TaskInstanceApi* | [**get_log**](docs/TaskInstanceApi.md#get_log) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/logs/{task_try_number} | Get logs
*TaskInstanceApi* | [**get_mapped_task_instance**](docs/TaskInstanceApi.md#get_mapped_task_instance) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/{map_index} | Get a mapped task instance
*TaskInstanceApi* | [**get_mapped_task_instances**](docs/TaskInstanceApi.md#get_mapped_task_instances) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/listMapped | List mapped task instances
*TaskInstanceApi* | [**get_task_instance**](docs/TaskInstanceApi.md#get_task_instance) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id} | Get a task instance
*TaskInstanceApi* | [**get_task_instances**](docs/TaskInstanceApi.md#get_task_instances) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances | List task instances
*TaskInstanceApi* | [**get_task_instances_batch**](docs/TaskInstanceApi.md#get_task_instances_batch) | **POST** /dags/~/dagRuns/~/taskInstances/list | List task instances (batch)
*UserApi* | [**delete_user**](docs/UserApi.md#delete_user) | **DELETE** /users/{username} | Delete a user
*UserApi* | [**get_user**](docs/UserApi.md#get_user) | **GET** /users/{username} | Get a user
*UserApi* | [**get_users**](docs/UserApi.md#get_users) | **GET** /users | List users
*UserApi* | [**patch_user**](docs/UserApi.md#patch_user) | **PATCH** /users/{username} | Update a user
*UserApi* | [**post_user**](docs/UserApi.md#post_user) | **POST** /users | Create a user
*VariableApi* | [**delete_variable**](docs/VariableApi.md#delete_variable) | **DELETE** /variables/{variable_key} | Delete a variable
*VariableApi* | [**get_variable**](docs/VariableApi.md#get_variable) | **GET** /variables/{variable_key} | Get a variable
*VariableApi* | [**get_variables**](docs/VariableApi.md#get_variables) | **GET** /variables | List variables
*VariableApi* | [**patch_variable**](docs/VariableApi.md#patch_variable) | **PATCH** /variables/{variable_key} | Update a variable
*VariableApi* | [**post_variables**](docs/VariableApi.md#post_variables) | **POST** /variables | Create a variable
*XComApi* | [**get_xcom_entries**](docs/XComApi.md#get_xcom_entries) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/xcomEntries | List XCom entries
*XComApi* | [**get_xcom_entry**](docs/XComApi.md#get_xcom_entry) | **GET** /dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/xcomEntries/{xcom_key} | Get an XCom entry


## Documentation For Models

 - [Action](docs/Action.md)
 - [ActionCollection](docs/ActionCollection.md)
 - [ActionCollectionAllOf](docs/ActionCollectionAllOf.md)
 - [ActionResource](docs/ActionResource.md)
 - [ClassReference](docs/ClassReference.md)
 - [ClearTaskInstance](docs/ClearTaskInstance.md)
 - [CollectionInfo](docs/CollectionInfo.md)
 - [Color](docs/Color.md)
 - [Config](docs/Config.md)
 - [ConfigOption](docs/ConfigOption.md)
 - [ConfigSection](docs/ConfigSection.md)
 - [Connection](docs/Connection.md)
 - [ConnectionAllOf](docs/ConnectionAllOf.md)
 - [ConnectionCollection](docs/ConnectionCollection.md)
 - [ConnectionCollectionAllOf](docs/ConnectionCollectionAllOf.md)
 - [ConnectionCollectionItem](docs/ConnectionCollectionItem.md)
 - [ConnectionTest](docs/ConnectionTest.md)
 - [CronExpression](docs/CronExpression.md)
 - [DAG](docs/DAG.md)
 - [DAGCollection](docs/DAGCollection.md)
 - [DAGCollectionAllOf](docs/DAGCollectionAllOf.md)
 - [DAGDetail](docs/DAGDetail.md)
 - [DAGDetailAllOf](docs/DAGDetailAllOf.md)
 - [DAGRun](docs/DAGRun.md)
 - [DAGRunCollection](docs/DAGRunCollection.md)
 - [DAGRunCollectionAllOf](docs/DAGRunCollectionAllOf.md)
 - [DagState](docs/DagState.md)
 - [Error](docs/Error.md)
 - [EventLog](docs/EventLog.md)
 - [EventLogCollection](docs/EventLogCollection.md)
 - [EventLogCollectionAllOf](docs/EventLogCollectionAllOf.md)
 - [ExtraLink](docs/ExtraLink.md)
 - [ExtraLinkCollection](docs/ExtraLinkCollection.md)
 - [HealthInfo](docs/HealthInfo.md)
 - [HealthStatus](docs/HealthStatus.md)
 - [ImportError](docs/ImportError.md)
 - [ImportErrorCollection](docs/ImportErrorCollection.md)
 - [ImportErrorCollectionAllOf](docs/ImportErrorCollectionAllOf.md)
 - [InlineResponse200](docs/InlineResponse200.md)
 - [InlineResponse2001](docs/InlineResponse2001.md)
 - [ListDagRunsForm](docs/ListDagRunsForm.md)
 - [ListTaskInstanceForm](docs/ListTaskInstanceForm.md)
 - [MetadatabaseStatus](docs/MetadatabaseStatus.md)
 - [PluginCollection](docs/PluginCollection.md)
 - [PluginCollectionAllOf](docs/PluginCollectionAllOf.md)
 - [PluginCollectionItem](docs/PluginCollectionItem.md)
 - [Pool](docs/Pool.md)
 - [PoolCollection](docs/PoolCollection.md)
 - [PoolCollectionAllOf](docs/PoolCollectionAllOf.md)
 - [Provider](docs/Provider.md)
 - [ProviderCollection](docs/ProviderCollection.md)
 - [RelativeDelta](docs/RelativeDelta.md)
 - [Resource](docs/Resource.md)
 - [Role](docs/Role.md)
 - [RoleCollection](docs/RoleCollection.md)
 - [RoleCollectionAllOf](docs/RoleCollectionAllOf.md)
 - [SLAMiss](docs/SLAMiss.md)
 - [ScheduleInterval](docs/ScheduleInterval.md)
 - [SchedulerStatus](docs/SchedulerStatus.md)
 - [Tag](docs/Tag.md)
 - [Task](docs/Task.md)
 - [TaskCollection](docs/TaskCollection.md)
 - [TaskExtraLinks](docs/TaskExtraLinks.md)
 - [TaskInstance](docs/TaskInstance.md)
 - [TaskInstanceCollection](docs/TaskInstanceCollection.md)
 - [TaskInstanceCollectionAllOf](docs/TaskInstanceCollectionAllOf.md)
 - [TaskInstanceReference](docs/TaskInstanceReference.md)
 - [TaskInstanceReferenceCollection](docs/TaskInstanceReferenceCollection.md)
 - [TaskState](docs/TaskState.md)
 - [TimeDelta](docs/TimeDelta.md)
 - [TriggerRule](docs/TriggerRule.md)
 - [UpdateDagRunState](docs/UpdateDagRunState.md)
 - [UpdateTaskInstancesState](docs/UpdateTaskInstancesState.md)
 - [User](docs/User.md)
 - [UserAllOf](docs/UserAllOf.md)
 - [UserCollection](docs/UserCollection.md)
 - [UserCollectionAllOf](docs/UserCollectionAllOf.md)
 - [UserCollectionItem](docs/UserCollectionItem.md)
 - [UserCollectionItemRoles](docs/UserCollectionItemRoles.md)
 - [Variable](docs/Variable.md)
 - [VariableAllOf](docs/VariableAllOf.md)
 - [VariableCollection](docs/VariableCollection.md)
 - [VariableCollectionAllOf](docs/VariableCollectionAllOf.md)
 - [VariableCollectionItem](docs/VariableCollectionItem.md)
 - [VersionInfo](docs/VersionInfo.md)
 - [WeightRule](docs/WeightRule.md)
 - [XCom](docs/XCom.md)
 - [XComAllOf](docs/XComAllOf.md)
 - [XComCollection](docs/XComCollection.md)
 - [XComCollectionAllOf](docs/XComCollectionAllOf.md)
 - [XComCollectionItem](docs/XComCollectionItem.md)


## Documentation For Authorization


## Basic

- **Type**: HTTP basic authentication


## Kerberos



## Author

dev@airflow.apache.org


## Notes for Large OpenAPI documents
If the OpenAPI document is large, imports in client.apis and client.models may fail with a
RecursionError indicating the maximum recursion limit has been exceeded. In that case, there are a couple of solutions:

Solution 1:
Use specific imports for apis and models like:
- `from airflow_client.client.api.default_api import DefaultApi`
- `from airflow_client.client.model.pet import Pet`

Solution 2:
Before importing the package, adjust the maximum recursion limit as shown below:
```
import sys
sys.setrecursionlimit(1500)
import airflow_client.client
from airflow_client.client.apis import *
from airflow_client.client.models import *
```

