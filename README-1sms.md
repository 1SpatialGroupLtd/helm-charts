# 1SMS Chart Settings

1SMS Deployment consists of 8 individual charts for the individual components :

* 1sms-common
* 1sms-1plan
* 1sms-1workflow-worklist
* 1sms-1exchange
* 1sms-1transact
* 1sms-1integrate-interface
* 1sms-1integrate-engine
* 1sms-job-orchestration
* 1sms-configuration

**Latest version 1.3.0**

## Pre-requisites

### Database Set up

The 1Spatial Management Suite components need access to an Oracle Database to store configuration data and connect to a Workspace Manager-enabled Oracle Spatial Feature Database.

Before installing the product, you must create the necessary users and schemas. The database locations, schema names, and passwords are needed to configure the helm charts.
Please refer to [1SMS Database Creation][] for further information.

### Temporal

The next-generation Job Orchestration service in the 1Spatial Management Suite utilises the Temporal orchestration engine to host 1SMS workflows.
[Temporal.io][] is a distributed, scalable, durable, and highly available orchestration engine, designed to execute asynchronous, long-running business logic in a resilient manner.

For optimal performance, 1SMS deployment requires a self-hosted Temporal instance, ideally on the same cluster as the 1SMS components.
Please utilise the Temporal official Helm charts for deployment [Temporal Helm Charts][].

_Note: Temporal requires its own repository which can be deployed as part of the helm chart. Please consult Temporal documentation._

## Authentication and Authorisation

### Authentication
1Spatial Management Suite is a secured system using an LDAP server to provide the authentication.
The services that make up 1Spatial Management Suite need to connect to the LDAP server using authentication providers.
The settings below are standard settings required to access an LDAP server, configured under the 1sms-common component
for all services. In addition, authorisation is provided by mapping LDAP user groups to 1SMS roles

```yaml
  authentication:
    ldap:
      host: <ldap_host>
      principal: <ldap_bind_principal>
      credential: <ldap_bind_credential>
      user_dn: <ldap_user_base_dn>
      group_dn: <ldap_group_base_dn>
    roleMapping:
          MyLdapGroup: 1SMS_admin,planner,worker,1int-admin,1int-user
          MyPlanners: planner
          MyAdmins: 1SMS_admin, planner, worker,1int-admin,1int-user
          MyWorkers: worker
          MyWorkersWithAccessTo1Integrate: worker,1int-user
    tls:
      enabled: false
```
Note: Ensure that the groups filter returns a reasonable amount of groups (ideally less than 50). For very large organisations or organisations who are part of a larger LDAP directory,
a suitable filter will ensure that only relevant groups are listed.

### Authorisation

1SMS creates roles for access to different levels of functionality for each product.
Each user who needs to use 1SMS must be allocated the appropriate roles.
The main roles required are :

* **1SMS_Admin**: User role with permissions to view and edit administration and configuration.
* **planner**: User role with permissions to view, create and edit 1SMS Jobs via 1Plan.
* **worker**: Minimal user role with permissions sufficient to run a 1SMS Job end to end via API or user interfaces.

For additional information on configuring 1SMS User Groups and Roles, please refer to [1SMS Documentation][].

1SMS roles will not grant Users access to configure 1Integrate. Standard roles required for 1Integrate are :
* **int_user**: Minimal user role with permissions to create, view and edit 1Integrate resources.
* **int_admin**: 1Integrate administration role.

For information on configuring 1Integrate Users and Roles, please refer to [1Integrate Documentation][].

## Required Common Settings

For each component the following settings are required :

| Parameter             | Description                                                                                                                                                                                                |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `version`             | The docker image tag to use, e.g `4.0.0`.                                                                                                                                                                  |
| `security.url`        | Full JDBC connection string for the Security schema. Note this is the same schema across all components. Please consult [1SMS Job Orchestration Service](#1sms-job-orchestration-service) for differences. |
| `security.username`   | Security schema database username.                                                                                                                                                                         |
| `security.password`   | Security schema database password.                                                                                                                                                                         |
| `repository.url`      | Full JDBC connection string for individual component repositories. Note 1Transact does not use this setting.                                                                                               |
| `repository.username` | Component repository database username.                                                                                                                                                                    |
| `repository.password` | Component repository database password.                                                                                                                                                                    |
| `opts`                | Additional system properties to pass into the components.                                                                                                                                                  |
| `temporal.host`       | Temporal frontend service hostname. Can be internal to the kubernetes cluster - e.g. temporal-frontend-headless.temporal.svc.cluster.local.                                                                |

### Feature Schema

1sms-transact, 1sms-exchange and 1sms-integrate-interface require connection to the underlying Oracle Workspace Manager enabled feature schema.

```yaml
    database:
      feature:
        url: jdbc:oracle:thin:@host:port/databaseName
        username: featureSchemaUsername
        password: featureSchemaPassword
```

### Component Specific Settings

#### 1Integrate

1Integrate is a licensed component and requires Base64 encoded 1Integrate licence file.
Licenses are provisioned by 1Spatial.

| Parameter                                | Description                                               | 
|------------------------------------------|-----------------------------------------------------------|
| `1sms-1integrate-engine.license.file`    | Base64 encoded 1Integrate license file.     |
| `1sms-1integrate-interface.license.file` | Base64 encoded 1Integrate license file.  |

##### Custom Extensions Settings (Optional)

1Integrate supports loading official extensions to support Custom Built-ins and Custom Datastores.
The setting allows extensions to be stored in external cloud storage such as Azure Blob/File storage or an S3 bucket.

This needs to be applied to both the interface and engine components.

```yaml
  extensions:
    claimName: integrate-extensions-blob
```
* **integrate-extensions-blob** - The name of an existing PersistentVolumeClaim to mount in as 1Integrate extensions.

#### 1Exchange

| Parameter                       | Description                                                               | Default                                                               |
|---------------------------------|---------------------------------------------------------------------------|-----------------------------------------------------------------------|
| `1sms-1exchange.config.baseUrl` | 1Exchange Rest interface baseUrl needed to define 1Exchange package paths | `https://MY_INGRESS_URL/1exchange/rest`.                              |

##### Conflict Resolution Name Mappings (Optional)

To support conflict resolution through GO Loader and GO Publisher, the conflict resolution name mappings XML files must be provided. For each file, both the filename and file content must be provided. The filename must follow the required naming convention `<product>_translationConfig.xml`. An example translation configuration:

```yaml
    translationConfig:
      - name: "MyProduct1_translationConfig.xml"
        content: |
          <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
          <TranslationSummary xmlns="http://www.snowflakesoftware.com/agent/translationConfig">
              <tableTranslation>
                  <table>TABLE1</table>
                  <element>
                      <name>Table1</name>
                      <namespace>http://www.example.org/myschema</namespace>
                  </element>
                  <primaryKeyColumn>ID</primaryKeyColumn>
                  <identifierElement>
                      <name>identifier</name>
                      <namespace>http://www.opengis.net/gml/3.2</namespace>
                  </identifierElement>
              </tableTranslation>
              <tableTranslation>
                  <table>TABLE2</table>
                  <element>
                      <name>Table2</name>
                      <namespace>http://www.example.org/myschema</namespace>
                  </element>
                  <primaryKeyColumn>ID</primaryKeyColumn>
                  <identifierElement>
                      <name>identifier</name>
                      <namespace>http://www.opengis.net/gml/3.2</namespace>
                  </identifierElement>
              </tableTranslation>
          </TranslationSummary>
      - name: "MyProduct2_translationConfig.xml"
        content: |
          <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
          <TranslationSummary xmlns="http://www.snowflakesoftware.com/agent/translationConfig">
              <tableTranslation>
                  <table>TABLE3</table>
                  <element>
                      <name>Table3</name>
                      <namespace>http://www.example.org/myschema</namespace>
                  </element>
                  <primaryKeyColumn>ID</primaryKeyColumn>
                  <identifierElement>
                      <name>identifier</name>
                      <namespace>http://www.opengis.net/gml/3.2</namespace>
                  </identifierElement>
              </tableTranslation>
              <tableTranslation>
                  <table>TABLE4</table>
                  <element>
                      <name>Table4</name>
                      <namespace>http://www.example.org/myschema</namespace>
                  </element>
                  <primaryKeyColumn>ID</primaryKeyColumn>
                  <identifierElement>
                      <name>identifier</name>
                      <namespace>http://www.opengis.net/gml/3.2</namespace>
                  </identifierElement>
              </tableTranslation>
          </TranslationSummary>
```
See [1SMS Documentation](https://1spatial.com/documentation/1SMS/v3_0/Topics/1Exchange/1EXC_Conflict_Res.htm) for more information.

#### 1SMS Job Orchestration Service

##### Security Schema Settings

The 1SMS Job Orchestration Service also requires connection to the 1SMS Security Schema, please note the difference in the jdbc url parameter name to other components :

```yaml
  database:
    security:
      jdbcUrl: jdbc:oracle:thin:@host:port/databaseName
      username: SMS_SECURITY
      password: SMS_SECURITY
```

##### Custom Workflows Settings (Optional)

1SMS Job Orchestration Service supports loading official extensions to support Custom Workflows.
The setting allows custom workflows to be stored in external cloud storage such as Azure Blob/File storage or an S3 bucket.


```yaml
  extensions:
    claimName: blob-my-workflow-extensions
```
* **blob-my-workflow-extensions** - The name of an existing PersistentVolumeClaim to mount in as Custom Workflow extensions.

##### 1SMS Configuration Service

1SMS Configuration service provides a single platform for configuring all 1SMS components for either specific application configuration or workflow orchestration.
The service is designed to work seamlessly in Kubernetes environments using config maps, eliminating the need for a separate database to share configuration across nodes.
1SMS Configuration services allows configuration changes to be applied without restarting services, allowing for faster iteration and reduced downtime.

###### Configuration Options Explained

The main sections of the configuration options are as following :

| Parameter          | Description                                                                                                   |
|--------------------|---------------------------------------------------------------------------------------------------------------|
| `map`              | Defines the spatial setup and available map layers to use in the planning user interface.                     |
| `gazetteer`        | Defines the configuration of location search and name-based queries in the planning user interface.           |
| `history`          | Enables tracking of job metadata changes over time. Disabled by default.                                      |
| `nameGeneration`   | Enables automatic name generation for jobs from a provided metadata. Disabled by default.                     |
| `metadata`         | Defines custom metadata definitions for jobs.                                                                 |
| `geometry`         | Defines the geometry extraction and submission handling policies used by 1Exchange.                           |
| *`workflow`        | Defines the orchestration logic for jobs.                                                                     |
| *`workflow.global` | Defines the orchestration logic inherited by all jobs types in the system.Can be overridden in workflow.types |
| *`workflow.types`  | Defines the custom orchestration logic per job type in the system.                                            |

**NOTE** : * Mandatory configuration settings. All other parameters are optional and should be configured.

###### Map

| Parameter                          | Description                                                                  |
|------------------------------------|------------------------------------------------------------------------------|
| `map.spatialReferenceSystem`       | The coordinate system used (e.g., `EPSG:2157` is Irish Transverse Mercator). |
| `map.initialExtent`                | The default bounding box shown when the map loads.                           |
| `map.maximumExtent`                | The full extent allowed for navigation or data display.                      |
| `map.wmsLayers`                    | List of Web Map Service layers.                                              |
| `map.wmsLayers.name`               | Display name of the layer.                                                   |
| `map.wmsLayers.url`                | WMS endpoint.                                                                |
| `map.wmsLayers.layer`              | Layer identifier on the server.                                              |
| `map.wmsLayers.format`             | Image format (e.g., PNG).                                                    |
| `map.wmsLayers.tiled`              | Whether the layer supports tiled rendering.                                  |
| `map.wfsLayers`                    | List of Web Feature Service layers.                                          |
| `map.wfsLayers.name`               | Display name.                                                                |
| `map.wfsLayers.url`                | WFS endpoint.                                                                |
| `map.wfsLayers.layer`              | Feature layer name.                                                          |
| `map.wfsLayers.geometryColumnName` | Property containing the geometry.                                            |
| `map.wmtsLayers`                   | List of Web Map Tile Service layers.                                         |
| `map.wmtsLayers.name`              | Display name.                                                                |
| `map.wmtsLayers.url`               | WMTS endpoint.                                                               |
| `map.wmtsLayers.matrixSet`         | Tile matrix set (usually matches the SRS).                                   |
| `map.wmtsLayers.layer`             | Layer identifier.                                                            |
| `map.wmtsLayers.format`            | Tile image format.                                                           |


###### Gazetteer

| Parameter                            | Description                                              |
|--------------------------------------|----------------------------------------------------------|
| `gazetteer.layer`                    | WFS layer used for gazetteer lookups.                    |
| `gazetteer.layer.url`                | WFS endpoint.                                            |
| `gazetteer.layer.name`               | Display name.                                            |
| `gazetteer.layer.layer`              | Layer name.                                              |
| `gazetteer.layer.geometryColumnName` | Property containing the geometry for spatial queries.    |
| `gazetteer.queryColumn`              | Property used for text-based search (e.g., place names). |

###### History

| Parameter         | Description                                                              |
|-------------------|--------------------------------------------------------------------------|
| `history.enabled` | Enables tracking of job metadata changes over time. Disabled by default. |

###### Name Generation

| Parameter                 | Description                                                                                                            |
|---------------------------|------------------------------------------------------------------------------------------------------------------------|
| `nameGeneration.enabled`  | Enabled automatic name generation for jobs from a provided metadata. Disabled by default.                              |
| `nameGeneration.template` | The template to generate a job name automatically if enabled.For example : "\${job.id}-\${job.type} \${custom.field1}" |

###### Metadata

| Parameter                                     | Description                                                          |
|-----------------------------------------------|----------------------------------------------------------------------|
| `metadata.customEnumerations`                 | Controlled vocabularies used for metadata classification.            |
| `metadata.customEnumerations.definition`      | Contains the ID and display name of the enumeration.                 |
| `metadata.customEnumerations.supportedValues` | List of allowed values for the enumeration.                          |
| `metadata.customFields`                       | Additional metadata fields that can be attached to features or jobs. |
| `metadata.customFields.id`                    | Field identifier used in the job metadata.                           |
| `metadata.customFields.displayName`           | Dispay name shown to users in user interface.                        |
| `metadata.customFields.dataType`              | Data type of the field (e.g., STRING).                               |
| `metadata.customFields.permission`            | Access level for the field (e.g., READ_WRITE).                       |
| `metadata.customFields.required`              | Indicates whether the field is mandatory.                            |
| `metadata.customJobStates`                    | Custom workflow states used in job processing.                       |
| `metadata.customJobStates.id`                 | State identifier to be used in orchestration processing.             |
| `metadata.customJobStates.displayName`        | Dispay name shown to users in user interface.                        |

###### Geometry

* **Snowflake Policy (Deprecated)**

Defines geometry handling policies using Snowflake configuration.

| Parameter                                             | Description                                                       |
|-------------------------------------------------------|-------------------------------------------------------------------|
| `geometry.snowflakePolicies.name`                     | Name of the Snowflake policy.                                     |
| `geometry.snowflakePolicies.srsName`                  | Spatial reference system used (e.g., `EPSG:4326`).                |
| `geometry.snowflakePolicies.growExtentsBuffer`        | Buffer applied to geometry extents.                               |
| `geometry.snowflakePolicies.extractionProject`        | Project name used during geometry extraction.                     |
| `geometry.snowflakePolicies.submissionProject`        | Project name used during geometry submission.                     |
| `geometry.snowflakePolicies.extractionBuffer`         | Buffer applied during geometry extraction.                        |
| `geometry.snowflakePolicies.extractionAdaptorEnabled` | Whether an adaptor is enabled for extraction.                     |
| `geometry.snowflakePolicies.submissionAdaptorEnabled` | Whether an adaptor is enabled for submission.                     |

*   Extraction adaptor parameters

If Extraction adaptor is enabled the following parameters need to be set

| Parameter                                                        | Description                                                     |
|------------------------------------------------------------------|-----------------------------------------------------------------|
| `geometry.snowflakePolicies.extractionAdaptor.fmeUrl`            | (If adapor enabled) URL of the FME service used for extraction. |
| `geometry.snowflakePolicies.extractionAdaptor.fmeUsername`       | Username for the FME extraction service.                        |
| `geometry.snowflakePolicies.extractionAdaptor.fmePassword`       | Password for the FME extraction service.                        |
| `geometry.snowflakePolicies.extractionAdaptor.fmeWorkspace`      | Workspace file used for extraction.                             |
| `geometry.snowflakePolicies.extractionAdaptor.overwriteFeatures` | Whether features should be overwritten during extraction.       |

* Submission adaptor parameters

If Submission adaptor is enabled the following parameters need to be set

| Parameter                                                   | Description                                 |
|-------------------------------------------------------------|---------------------------------------------|
| `geometry.snowflakePolicies.submissionAdaptor.fmeUrl`       | URL of the FME service used for submission. |
| `geometry.snowflakePolicies.submissionAdaptor.fmeUsername`  | Username for the FME submission service.    |
| `geometry.snowflakePolicies.submissionAdaptor.fmePassword`  | Password for the FME submission service.    |
| `geometry.snowflakePolicies.submissionAdaptor.fmeWorkspace` | Workspace file used for submission.         |

* **FME Policy**

Defines geometry handling policies using FME configuration.

| Parameter                                  | Description                                                 |
|--------------------------------------------|-------------------------------------------------------------|
| `geometry.fmePolicies.name`                | Name of the FME policy.                                     |
| `geometry.fmePolicies.url`                 | URL of the FME service.                                     |
| `geometry.fmePolicies.srsName`             | Spatial reference system used.                              |
| `geometry.fmePolicies.username`            | Username for the FME service.                               |
| `geometry.fmePolicies.password`            | Password for the FME service.                               |
| `geometry.fmePolicies.extractionWorkspace` | Workspace file used for extraction.                         |
| `geometry.fmePolicies.submissionWorkspace` | Workspace file used for submission.                         |

* **No-op Policy**

Defines basic No-Op geometry policies.

| Parameter                                  | Description                                     |
|--------------------------------------------|-------------------------------------------------|
| `geometry.noopPolicies.name`               | Name of the Noop policy.                        |
| `geometry.noopPolicies.srsName`            | Spatial reference system used.                  |
| `geometry.noopPolicies.growExtentsBuffer`  | Buffer applied to geometry extents.             |
| `geometry.noopPolicies.growExtentsColumns` | List of columns used to grow extents.           |

##### Workflow

| Parameter                                                          | Description                                                                                           | Example                                                                                          |
|--------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| * `workflow.global.jobIdColumn`                                    | The column in the feature database tables that will be populated with the 1SMS Job ID on update.      | `JOB_ID`                                                                                         |
| * `workflow.global.workflowType`                                   | Type of workflow engine used.                                                                         | `SmsOnlineEditingWorkflow` or `SmsOfflineEditingWorkflow`                                        |
| `geometryService.policyName`                                       | The geometry configuration name to use for this job type (e.g. the data and data formats to exchange) | Name of the geometry policy to use. Mandatory for SmsOfflineEditingWorkflow                      |
| `geometryService.policyName.growExtents.enabled`                   | Automatically grow the extent of a job.                                                               | `true` or `false`                                                                                |
| `geometryService.includeDiffViews`                                 | Include Diff Views With Conflict Extraction.                                                          | `true` or `false`                                                                                |
| * `workflow.global.validation.enabled`                             | Turns validation on or off.                                                                           | `true` or `false`                                                                                |
| `workflow.global.validation.rulesbase`                             | Validation ruleset to apply.                                                                          | Name of the business rules folder in 1Integrate contained within Production folder               |
| `workflow.global.validation.datastore`                             | Source of validation data.                                                                            | Name of the data store connecting to the source in 1Integrate contained within Production folder |
| `workflow.global.validation.refreshEnabled`                        | Whether validation refreshes automatically.                                                           | `true` or `false`                                                                                |
| `workflow.global.validation.assignment`                            | Controls automatic allocation of validation failure jobs.                                             | One of the assignment rules `WORKER`,`WORKER_GROUP` or `NAMED_GROUP`                             |
| `workflow.global.validation.assignment.validationGroup`            | Active Directory group if NAMED_GROUP automatic allocation is used.                                   | Name of the active directory group to address validation failures.                               |
| * `workflow.global.quarantine.enabled`                             | Enables quarantine feature.                                                                           | `true` or `false`                                                                                |
| `workflow.global.quarantine.assignment`                            | Controls automatic allocation of quarantined jobs.                                                    | One of the assignment rules `WORKER`,`WORKER_GROUP`,`NAMED_GROUP` or `METADATA`                  |
| `workflow.global.quarantine.assignment.reviewerGroup`              | Active Directory group if NAMED_GROUP automatic allocation is used.                                   | Name of the active directory group to address quarantine review.                                 |
| * `workflow.global.conflict.automaticResolution`                   | Enables or disables automatic conflict resolution.                                                    | `true` or `false`                                                                                |
| * `workflow.global.conflict.automaticResolution.maxRetries`        | Maximum number of attempts to resolve conflict automatically                                          | A whole number, e.g. `2`                                                                         |
| `workflow.global.conflict.automaticResolution.latestWinnerColumns` | Automatically resolve conflicts using the latest changes for the following columns                    | Column names in list format <br/>`latestWinnerColumns:` <br/> `- COL_1` <br/>`- COL_2`           |
| `workflow.global.conflict.assignment`                              | Controls automatic allocation of conflicted jobs.                                                     | One of the assignment rules `WORKER`,`WORKER_GROUP` or `NAMED_GROUP`                             |
| `workflow.global.conflict.assignment.conflictGroup`                | Active Directory group if NAMED_GROUP automatic allocation is used.                                   | Name of the active directory group to address conflict failures.                                 |
| * `workflow.global.packageManager.uploadErrorCleanup`              | Automatically cleans up failed uploads.                                                               | `true` or `false`                                                                                |
| * `workflow.types.TYPE_ID`                                         | Mandatory.Configuration for the job types configured in the system.                                   | Job type id , e.g. `FIELD_EDIT`                                                                  |
| * `workflow.types.TYPE_ID.displayName`                             | Mandatory.Display name for defined job type in the system.                                            | Display name of the job type, e.g. `Field Edit`                                                  |

**NOTE** : * Mandatory configuration settings. All other parameters are optional and should be configured if options enabled.
**NOTE** : All Globally defined workflow settings are inherited by default by all job types but can be overridden.

#### Email Notifications

Email Notifications can be configured in the system.

| Parameter                                                            | Description                                                                                                                                |
|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `workflow.notification.enabled`                                      | Enables or disables notifications globally,`true` or `false`                                                                               |
| `workflow.notification.stateConfiguration`                           | Defines notification behavior for specific workflow states.                                                                                |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyWorker`   | Whether to notify the assigned worker when a job is has transitioned to a given `JOB_STATUS`.                                              |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyPlanner`  | Whether to notify the planner when a job has transitioned to a given `JOB_STATUS`.                                                         |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyGroup`    | Whether to notify the worker group when a job has transitioned to a given `JOB_STATUS`.                                                    |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyReviewer` | Whether to notify the assigned reviewer when a job has transitioned to a given `JOB_STATUS`.                                               |
| `workflow.notification.admin.enabled`                                | Enables or disables admin notifications.                                                                                                   |
| `workflow.notification.admin.addresses`                              | List of email addresses to receive admin notifications, e.g.<br/>`addresses:` <br/> `- "admin1@example.org"` <br/>`- "admin2@example.org"` |

###### Connecting 1Plan to 1SMS Configuration Service

To ensure 1Plan is connected to the configuration service and configuration can be synchronised
the following

| Parameter                             | Description                                            |
|---------------------------------------|--------------------------------------------------------|
| `config.configurationServiceBaseUrl`  | The URL for the location of the Configuration Service. |

###### Connecting 1Exchange to 1SMS Configuration Service

To ensure 1Exchange is connected to the configuration service and configuration can be synchronised
the following

| Parameter                             | Description                                            |
|---------------------------------------|--------------------------------------------------------|
| `config.configurationServiceBaseUrl`  | The URL for the location of the Configuration Service. |

## Optional Recommended Settings

### Resources and replication settings

Resources and replications settings need to be tuned according to each environment needs and usages.

| Parameter   | Description                                               | Default                                  |
|-------------|-----------------------------------------------------------|------------------------------------------|
| `count`     | The number of replicas to deploy.                         | `1`.                                     |
| `memory`    | The heap space value to make available to each component. | No default value. Example : `1g`.        |
| `resources` | Configurable [resources][] for the each deployment.       | Memory requests and limits both `2048M`. |

_Note: For Production Environments the recommendation is to ensure Guaranteed Quality of Service for each component as per the recommended Kubernetes practices for [QoS][]._

_Note: Recommended number is a minimum of 2 replicas of 1sms-integrate-engine as that would allow for parallel validation sessions. This does depend on the 1Integrate license, please confirm with 1Spatial if you are licensed to run multiple engines._




### Ingress

| Parameter         | Description                                                | Default |
|-------------------|------------------------------------------------------------|---------|
| `ingress.enabled` | Whether 1SMS components need to be exposed via an ingress. | `true`. |
| `ingress.host`    | The host that will be used to access each 1SMS component.  | ``.     |

### TLS

| Parameter     | Description                                                                   | Default |
|---------------|-------------------------------------------------------------------------------|---------|
| `tls.enabled` | Whether to deploy with a TLS configuration, port 80 will be exposed if false. | `true`. |
| `tls.key`     | Base64 encoded TLS certificate key, can be for a self-signed certificate.     |         |
| `tls.crt`     | Base64 encoded TLS certificate, can be a self-signed certificate.             |         |

### Images and container registries

| Parameter          | Description                                                                         | Default         |
|--------------------|-------------------------------------------------------------------------------------|-----------------|
| `image.registry`   | The registry where the 1SMS docker images can be found, _without_ a trailing slash. | `Nil`.          |
| `image.pullPolicy` | [Kubernetes container pull policy][]                                                | `IfNotPresent`. |
| `imagePullSecrets` | [Kubernetes image pull secrets][]                                                   | `Nil`.          |

### Trusting Additional SSL Certificates

Each 1SMS Application can trust custom Certificate Authorities(CAs) by using certificates stored in an existing Kubernetes Secret.

| Parameter                    | Description                                                    | Default |
|------------------------------|----------------------------------------------------------------|---------|
| `trustedCertsExistingSecret` | Existing Kubernetes Secret containing additional certificates. | `Nil`.  |


[Temporal.io]: https://temporal.io/
[Temporal Helm Charts]: https://github.com/temporalio/helm-charts
[1SMS Database Creation]: https://1spatial.com/documentation/1SMS/v3_0/Topics/1SMS_Database_Creation.htm
[1SMS Documentation]: https://1spatial.com/documentation/1SMS
[1Integrate Documentation]: https://1spatial.com/documentation/1integrate
[resources]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container
[QoS]: https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
[Kubernetes container pull policy]: https://kubernetes.io/docs/concepts/containers/images/#updating-images
[Kubernetes image pull secrets]: https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets

Copyright (c) 1Spatial 2025
