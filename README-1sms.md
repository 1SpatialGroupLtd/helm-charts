# 1SMS Chart Settings

1SMS Deployment consists of 9 individual charts for the individual components:

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

The next-generation Job Orchestration service in the 1Spatial Management Suite utilises the Temporal orchestration engine to manage 1SMS workflows.
[Temporal.io][] is a distributed, scalable, durable, and highly available orchestration engine, designed to execute asynchronous, long-running business logic in a resilient manner.

For optimal performance, 1SMS deployment requires a self-hosted Temporal instance, ideally on the same cluster as the 1SMS components.
Please utilise the Temporal official Helm charts for deployment [Temporal Helm Charts][].

_Note: Temporal requires its own repository which can be deployed as part of the helm chart. Please consult Temporal documentation._

## Authentication and Authorisation

### Authentication
1Spatial Management Suite is a secured system using an LDAP server to provide the authentication.
The services that make up 1Spatial Management Suite need to connect to the LDAP server using authentication providers.
The settings below are standard settings required to access an LDAP server, configured under the 1sms-common component for all services. In addition, authorisation is provided by mapping LDAP user groups to 1SMS roles

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
The main roles required are:

* **1SMS_Admin**: User role with permissions to view and edit administration and configuration.
* **planner**: User role with permissions to view, create and edit 1SMS Jobs via 1Plan.
* **worker**: Minimal user role with permissions sufficient to run a 1SMS Job end to end via API or user interfaces.

For additional information on configuring 1SMS User Groups and Roles, please refer to [1SMS Documentation][].

1SMS roles will not grant Users access to configure 1Integrate. Standard roles required for 1Integrate are:
* **int_user**: Minimal user role with permissions to create, view and edit 1Integrate resources.
* **int_admin**: 1Integrate administration role.

For information on configuring 1Integrate Users and Roles, please refer to [1Integrate Documentation][].

## Required Common Settings

For each component the following settings are required:

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

1Integrate is a licensed component and requires a Base64 encoded 1Integrate licence file.
Licenses are provisioned by 1Spatial.

| Parameter                                | Description                             | 
|------------------------------------------|-----------------------------------------|
| `1sms-1integrate-engine.license.file`    | Base64 encoded 1Integrate license file. |
| `1sms-1integrate-interface.license.file` | Base64 encoded 1Integrate license file. |

##### Custom Extensions Settings (Optional)

1Integrate supports loading official extensions to support Custom Built-ins and Custom Data Stores.
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

The 1SMS Job Orchestration Service also requires the connection to the 1SMS Security Schema, please note the difference in the jdbc url parameter name to other components:

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
* **blob-my-workflow-extensions** - The name of an existing PersistentVolumeClaim to mount in containing Custom Workflow extensions.

#### 1SMS Configuration Service

1SMS Configuration Service provides a means for configuring 1SMS from a single place. This covers both specific application configuration or workflow runtime settings.
Configuration changes can be applied to the system without needing to restart services, allowing for faster iteration and reduced downtime.

Note: Running workflows will not pick up configuration updates. These will only be applied to new jobs started after the configuration change.

##### Configuration Options

The main sections of the configuration options are as following:

| Parameter          | Description                                                                                                                                        |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `map`              | Defines the spatial setup and available map layers to use in the planning user interface.                                                          |
| `gazetteer`        | Defines the configuration of location search and name-based queries in the planning user interface.                                                |
| `history`          | Enables tracking of job metadata changes over time. Disabled by default.                                                                           |
| `nameGeneration`   | Enables automatic name generation for jobs using metadata fields from the provided template. Disabled by default.                                  |
| `metadata`         | Defines custom metadata definitions for jobs.                                                                                                      |
| `geometry`         | Defines the geometry extraction and submission handling policies used by 1Exchange.                                                                |
| *`workflow`        | Defines the orchestration logic for jobs.                                                                                                          |
| *`workflow.global` | Defines the orchestration logic used as a base for all jobs types in the system. Specific values can be overridden in individual `workflow.types`. |
| *`workflow.types`  | Defines the custom orchestration logic per job type in the system.                                                                                 |

**NOTE** : * Mandatory configuration settings

##### Map

Configuration to support the map view of the 1SMS User Interface.

| Parameter                          | Description                                                                            |
|------------------------------------|----------------------------------------------------------------------------------------|
| `map.spatialReferenceSystem`       | The coordinate system to use when displaying the map and job tiles (e.g. `EPSG:4326`). |
| `map.initialExtent`                | The default map extent shown to new users when they first open the map.                |
| `map.maximumExtent`                | The full extent allowed for navigation or data display.                                |
| `map.wmsLayers`                    | List of Web Map Service layers.                                                        |
| `map.wmsLayers.name`               | Display name of the layer.                                                             |
| `map.wmsLayers.url`                | WMS endpoint.                                                                          |
| `map.wmsLayers.layer`              | Layer identifier on the server.                                                        |
| `map.wmsLayers.format`             | Image format (e.g. `JPEG` or `PNG`).                                                   |
| `map.wmsLayers.tiled`              | Whether the layer supports tiled rendering.                                            |
| `map.wfsLayers`                    | List of Web Feature Service layers.                                                    |
| `map.wfsLayers.name`               | Display name.                                                                          |
| `map.wfsLayers.url`                | WFS endpoint.                                                                          |
| `map.wfsLayers.layer`              | Feature layer name.                                                                    |
| `map.wfsLayers.geometryColumnName` | Property containing the geometry.                                                      |
| `map.wfsLayers.featureNamespace`   | The namespace URI for features in the layer.                                           |
| `map.wfsLayers.featurePrefix`      | The prefix for the feature namespace.                                                  |
| `map.wmtsLayers`                   | List of Web Map Tile Service layers.                                                   |
| `map.wmtsLayers.name`              | Display name.                                                                          |
| `map.wmtsLayers.url`               | WMTS endpoint.                                                                         |
| `map.wmtsLayers.matrixSet`         | Tile matrix set.                                                                       |
| `map.wmtsLayers.layer`             | Layer identifier.                                                                      |
| `map.wmtsLayers.format`            | Tile image format.                                                                     |

Example configuration:

```yaml
map:
  spatialReferenceSystem: EPSG:4626
  initialExtent:
    minX: -9.01
    minY: 49.75
    maxX: 2.01
    maxY: 61.01
  maximumExtent:
    minX: -180.0
    minY: -90.0
    maxX: 180.0
    maxY: 90.0
  wmsLayers:
    - name: Layer A
      url: https://example.com/wms
      layer: layerA
      format: PNG
      tiled: true
    - name: Layer B
      url: https://example.com/wms
      layer: layerB
      format: JPEG
      tiled: false
  wmtsLayers:
    - name: Layer C
      url: https://example.com/wmts
      matrixSet: EPSG:4626
      layer: layerC
      format: PNG
  wfsLayers:
    - url: https://example.com/wfs
      name: Layer F
      layer: layerF
      geometryColumnName: GEOM
      featureNamespace: https://example.com/ns
      featurePrefix: test-ns
    - url: https://example.com/wfs
      name: Layer G
      layer: layerG
      geometryColumnName: GEOM2
      featureNamespace: https://example.com/ns
      featurePrefix: test2-ns
```

##### Gazetteer

Configuration to support gazetteer search within the map view of the 1SMS User Interface.

| Parameter                            | Description                                             |
|--------------------------------------|---------------------------------------------------------|
| `gazetteer.layer`                    | WFS layer used for gazetteer lookups.                   |
| `gazetteer.layer.url`                | WFS endpoint.                                           |
| `gazetteer.layer.name`               | Display name.                                           |
| `gazetteer.layer.layer`              | Layer name.                                             |
| `gazetteer.layer.geometryColumnName` | Property containing the geometry for spatial queries.   |
| `gazetteer.layer.featureNamespace`   | The namespace URI for features in the layer.            |
| `gazetteer.layer.featurePrefix`      | The prefix for the feature namespace.                   |
| `gazetteer.queryColumn`              | Property used for text-based search (e.g. place names). |

Example configuration:

```yaml
gazetteer:
  layer:
    url: https://example.com/gazetteer
    name: Gazetteer
    layer: gazetteer
    geometryColumnName: GEOM
    featureNamespace: https://example.com/gazetteer
    featurePrefix: gazetteer-ns
  queryColumn: NAME
```

##### History

| Parameter         | Description                                                              |
|-------------------|--------------------------------------------------------------------------|
| `history.enabled` | Enables tracking of job metadata changes over time. Disabled by default. |

##### Name Generation

| Parameter                 | Description                                                                                                                                                                                                                                    |
|---------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `nameGeneration.enabled`  | Enabled automatic name generation for jobs from a provided metadata. Disabled by default.                                                                                                                                                      |
| `nameGeneration.template` | The template to generate a job name from. The template can contain text and references to job metadata fields, both system and custom. Custom fields should be prefixed `custom.` For example: `Job \${job.id}-\${job.type} \${custom.field1}` |

Example configuration:

```yaml
nameGeneration:
  enabled: true
  template: "${job.id}-${job.type}"
```

##### Metadata

Allows for custom job metadata fields and statuses to be defined in addition to the standard system metadata.

| Parameter                                     | Description                                                          |
|-----------------------------------------------|----------------------------------------------------------------------|
| `metadata.customEnumerations`                 | Controlled vocabularies used for metadata classification.            |
| `metadata.customEnumerations.definition`      | Contains the ID and display name of the enumeration.                 |
| `metadata.customEnumerations.supportedValues` | List of allowed values for the enumeration.                          |
| `metadata.customFields`                       | Additional metadata fields that can be attached to features or jobs. |
| `metadata.customFields.id`                    | Field identifier used in the job metadata.                           |
| `metadata.customFields.displayName`           | Display name shown to users in user interface.                       |
| `metadata.customFields.dataType`              | Data type of the field (e.g. `STRING`).                              |
| `metadata.customFields.permission`            | Access level for the field (e.g. `READ_WRITE`).                      |
| `metadata.customFields.required`              | Indicates whether the field is mandatory.                            |
| `metadata.customJobStates`                    | Custom workflow states used in job processing.                       |
| `metadata.customJobStates.id`                 | State identifier to be used in orchestration processing.             |
| `metadata.customJobStates.displayName`        | Display name shown to users in user interface.                       |

Example configuration:

```yaml
metadata:
  customEnumerations:
    - definition:
        id: MY_ENUM
        displayName: My Enum
      supportedValues:
        - id: VALUE_1
          displayName: Value 1
        - id: VALUE_2
          displayName: Value 2
        - id: VALUE_3
          displayName: Value 3
  customFields:
    - id: field1
      displayName: Field 1
      dataType: STRING
      permission: READ_WRITE
      required: false
    - id: field2
      displayName: Field 2
      dataType: ENUMERATION
      permission: READ_WRITE
      required: true
      enumeration: MY_ENUM
    - id: field3
      displayName: Field 3
      dataType: STRING
      permission: READ_ONLY
      required: false
  customJobStates:
    - id: MY_STATE_1
      displayName: My Status 1
    - id: MY_STATE_2
      displayName: My Status 2
```      

##### Geometry

Configuration of geometry extraction and submission processes.

###### Snowflake Policy (Deprecated)

Defines geometry handling policies using Snowflake configuration.

| Parameter                                             | Description                                                      |
|-------------------------------------------------------|------------------------------------------------------------------|
| `geometry.snowflakePolicies.name`                     | Name of the Snowflake policy.                                    |
| `geometry.snowflakePolicies.srsName`                  | Spatial reference system of the feature data (e.g. `EPSG:4326`). |
| `geometry.snowflakePolicies.growExtentsBuffer`        | Buffer applied to grown geometry extents.                        |
| `geometry.snowflakePolicies.extractionProject`        | Project name to use for geometry extraction.                     |
| `geometry.snowflakePolicies.submissionProject`        | Project name to use for geometry submission.                     |
| `geometry.snowflakePolicies.extractionBuffer`         | Buffer applied during geometry extraction.                       |
| `geometry.snowflakePolicies.extractionAdaptorEnabled` | Whether an adaptor is enabled for extraction.                    |
| `geometry.snowflakePolicies.submissionAdaptorEnabled` | Whether an adaptor is enabled for submission.                    |

*   Extraction Adaptor Parameters

If an Extraction Adaptor is required to perform an additional GML translation, configure as follows:

| Parameter                                                        | Description                                               |
|------------------------------------------------------------------|-----------------------------------------------------------|
| `geometry.snowflakePolicies.extractionAdaptor.fmeUrl`            | URL of the FME service to use during extraction.          |
| `geometry.snowflakePolicies.extractionAdaptor.fmeUsername`       | Username for this FME service.                            |
| `geometry.snowflakePolicies.extractionAdaptor.fmePassword`       | Password for this FME service.                            |
| `geometry.snowflakePolicies.extractionAdaptor.fmeWorkspace`      | FME Workspace to use during extraction.                   |
| `geometry.snowflakePolicies.extractionAdaptor.overwriteFeatures` | Whether features should be overwritten during extraction. |

* Submission Adaptor Parameters

If a Submission Adaptor is required to perform an additional GML translation, configure as follows:

| Parameter                                                   | Description                                      |
|-------------------------------------------------------------|--------------------------------------------------|
| `geometry.snowflakePolicies.submissionAdaptor.fmeUrl`       | URL of the FME service to use during submission. |
| `geometry.snowflakePolicies.submissionAdaptor.fmeUsername`  | Username for the this service.                   |
| `geometry.snowflakePolicies.submissionAdaptor.fmePassword`  | Password for the this service.                   |
| `geometry.snowflakePolicies.submissionAdaptor.fmeWorkspace` | FME Workspace to use during submission.          |

Example configuration:

```yaml
geometry:
  snowflakePolicies:
    - name: Snowflake
      srsName: EPSG:4326
      growExtentsBuffer: 0.0
      extractionProject: Extract_Project
      submissionProject: Submit_Project
      extractionBuffer: 0.0
    - name: Snowflake-Adapted
      srsName: EPSG:4326
      growExtentsBuffer: 0.0
      extractionProject: Extract_Project2
      submissionProject: Submit_Project2
      extractionBuffer: 10.0
      extractionAdaptorEnabled: true
      submissionAdaptorEnabled: true
      extractionAdaptor:
        fmeUrl: https://fme-service1.com
        fmeUsername: admin1
        fmePassword: password1
        fmeWorkspace: extract_adapt_workspace.fmw
        overwriteFeatures: false
      submissionAdaptor:
        fmeUrl: https://fme-service2.com
        fmeUsername: admin2
        fmePassword: password2
        fmeWorkspace: submit_adapt_workspace.fmw
```

###### FME Policy

Defines geometry handling policies using FME configuration.

| Parameter                                  | Description                                   |
|--------------------------------------------|-----------------------------------------------|
| `geometry.fmePolicies.name`                | Name of the FME policy.                       |
| `geometry.fmePolicies.url`                 | URL of the FME service.                       |
| `geometry.fmePolicies.srsName`             | Spatial reference system of the feature data. |
| `geometry.fmePolicies.growExtentsBuffer`   | Buffer applied to grown geometry extents.     |
| `geometry.fmePolicies.growExtentsColumns`  | List of columns used to grow extents.         |
| `geometry.fmePolicies.username`            | Username for the FME service.                 |
| `geometry.fmePolicies.password`            | Password for the FME service.                 |
| `geometry.fmePolicies.extractionWorkspace` | Workspace file used for extraction.           |
| `geometry.fmePolicies.submissionWorkspace` | Workspace file used for submission.           |

Example configuration:

```yaml
geometry:
  fmePolicies:
    - name: FME1
      url: "https://fme-service.com"
      srsName: EPSG:1234
      username: myusername
      password: mypassword
      extractionWorkspace: extraction_workspace.fmw
      submissionWorkspace: submission_workspace.fmw
    - name: FME2
      url: "https://fme-service.com"
      srsName: EPSG:4326
      username: myusername
      password: mypassword
      extractionWorkspace: extraction_workspace2.fmw
      submissionWorkspace: submission_workspace2.fmw
```

###### No-op Policy

Defines basic No-Op geometry policies.

| Parameter                                  | Description                                   |
|--------------------------------------------|-----------------------------------------------|
| `geometry.noopPolicies.name`               | Name of the No-op policy.                     |
| `geometry.noopPolicies.srsName`            | Spatial reference system of the feature data. |
| `geometry.noopPolicies.growExtentsBuffer`  | Buffer applied to grown geometry extents.     |
| `geometry.noopPolicies.growExtentsColumns` | List of columns used to grow extents.         |

Example configuration:

```yaml
geometry:
  noopPolicies:
  - name: Noop1
    srsName: EPSG:4326
    growExtentsBuffer: 100.5
    growExtentsColumns:
      - column1
      - column2
  - name: Noop2
    srsName: EPSG:1234
```

##### Workflow

Runtime workflow configuration for 1SMS jobs.

| Parameter                                                          | Description                                                                                                                                  | Example                                                                                          |
|--------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| * `workflow.global.jobIdColumn`                                    | The column in the feature database tables that will be populated with the 1SMS Job ID on update.                                             | `JOB_ID`                                                                                         |
| * `workflow.global.workflowType`                                   | Type of workflow to run for this job type. Available types are listed by the Job Orchestration Service API.                                  | `SmsOnlineEditingWorkflow` or `SmsOfflineEditingWorkflow`                                        |
| `geometryService.policyName`                                       | The geometry configuration name to use for this job type (e.g. the data and data formats to exchange)                                        | Name of the geometry policy to use. Mandatory for SmsOfflineEditingWorkflow                      |
| `geometryService.policyName.growExtents.enabled`                   | Whether to automatically grow the extent of a job to ensure that any features partially included will be wholly within the area of interest. | `true` or `false`                                                                                |
| `geometryService.includeDiffViews`                                 | Include Diff Views With Conflict Extraction.                                                                                                 | `true` or `false`                                                                                |
| * `workflow.global.validation.enabled`                             | Turns validation on or off.                                                                                                                  | `true` or `false`                                                                                |
| `workflow.global.validation.rulesbase`                             | Validation ruleset to apply.                                                                                                                 | Name of the business rules folder in 1Integrate contained within Production folder               |
| `workflow.global.validation.datastore`                             | Source of validation data.                                                                                                                   | Name of the data store connecting to the source in 1Integrate contained within Production folder |
| `workflow.global.validation.refreshEnabled`                        | Whether validation refreshes automatically.                                                                                                  | `true` or `false`                                                                                |
| `workflow.global.validation.assignment`                            | Controls automatic allocation of validation failure jobs.                                                                                    | One of the assignment rules `WORKER`,`WORKER_GROUP` or `NAMED_GROUP`                             |
| `workflow.global.validation.assignment.validationGroup`            | Active Directory group if `NAMED_GROUP` automatic allocation is used.                                                                        | Name of the active directory group to address validation failures.                               |
| * `workflow.global.quarantine.enabled`                             | Enables quarantine feature.                                                                                                                  | `true` or `false`                                                                                |
| `workflow.global.quarantine.assignment`                            | Controls automatic allocation of quarantined jobs.                                                                                           | One of the assignment rules `WORKER`,`WORKER_GROUP`,`NAMED_GROUP` or `METADATA`                  |
| `workflow.global.quarantine.assignment.reviewerGroup`              | Active Directory group if `NAMED_GROUP` automatic allocation is used.                                                                        | Name of the active directory group to address quarantine review.                                 |
| * `workflow.global.conflict.automaticResolution`                   | Enables or disables automatic conflict resolution.                                                                                           | `true` or `false`                                                                                |
| * `workflow.global.conflict.automaticResolution.maxRetries`        | Maximum number of attempts to resolve conflict automatically.                                                                                | A whole number, e.g. `2`                                                                         |
| `workflow.global.conflict.automaticResolution.latestWinnerColumns` | Automatically resolve conflicts using the latest changes for the following columns                                                           | Column names in list format <br/>`latestWinnerColumns:` <br/> `- COL_1` <br/>`- COL_2`           |
| `workflow.global.conflict.assignment`                              | Controls automatic allocation of conflicted jobs.                                                                                            | One of the assignment rules `WORKER`,`WORKER_GROUP` or `NAMED_GROUP`                             |
| `workflow.global.conflict.assignment.conflictGroup`                | Active Directory group if `NAMED_GROUP` automatic allocation is used.                                                                        | Name of the active directory group to address conflict failures.                                 |
| * `workflow.global.packageManager.uploadErrorCleanup`              | Whether to cleans up the uploaded change file in the event of a system failure.                                                              | `true` or `false`                                                                                |
| * `workflow.types.TYPE_ID`                                         | Mandatory. Configuration for the job types configured in the system.                                                                         | Job type id , e.g. `'MY_TYPE_1'`                                                                 |
| * `workflow.types.TYPE_ID.displayName`                             | Mandatory. Display name for defined job type in the system.                                                                                  | Display name of the job type, e.g. `My Type 1`                                                   |

**NOTE** : * Mandatory configuration settings.
**NOTE** : All Globally defined workflow settings are inherited by default by all job types but can be overridden.

###### Email Notifications

Configures when and if email notifications are to be sent to users based on job state changes or system errors.

| Parameter                                                            | Description                                                                                                                                |
|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `workflow.notification.enabled`                                      | Whether the system should send emails. Requires the 1SMS Job Orchestration Service to be configured with the relevant mail configuration.  |
| `workflow.notification.stateConfiguration`                           | Defines notification behavior for specific workflow states. This can apply to system and custom states alike.                              |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyWorker`   | Whether to notify the assigned worker when a job is has transitioned to a given `JOB_STATUS`.                                              |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyPlanner`  | Whether to notify the planner when a job has transitioned to a given `JOB_STATUS`.                                                         |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyGroup`    | Whether to notify the worker group when a job has transitioned to a given `JOB_STATUS`.                                                    |
| `workflow.notification.stateConfiguration.JOB_STATUS.notifyReviewer` | Whether to notify the assigned reviewer when a job has transitioned to a given `JOB_STATUS`.                                               |
| `workflow.notification.admin.enabled`                                | Whether emails will be sent to chosen administrators in the event of system errors.                                                        |
| `workflow.notification.admin.addresses`                              | List of email addresses to receive admin notifications, e.g.<br/>`addresses:` <br/> `- "admin1@example.org"` <br/>`- "admin2@example.org"` |

Example configuration:

```yaml
workflow:
  global:
    jobIdColumn: "JOB_ID"
    workflowType: "SmsOfflineEditingWorkflow"
    geometry:
      policyName: "MyPolicy"
      growExtents:
        enabled: true
      includeDiffViews: false
    validation:
      enabled: true
      rulesbase: "MyRules"
      datastore: "ValidationDataStore"
      refreshEnabled: false
      assignment: WORKER_GROUP
      validationGroup: "MyValidationGroup"
    quarantine:
      enabled: false
      assignment: METADATA
      reviewerGroup: "MyReviewers"
    conflict:
      automaticResolution:
        enabled: true
        maximumRetries: 0
        latestWinnerColumns:
          - column1
          - column2
      assignment: WORKER_GROUP
      conflictGroup: "MyConflictGroup"
    packageManager:
      uploadErrorCleanup: true
    notification:
      enabled: true
      stateConfiguration:
        ALLOCATED:
          notifyWorker: true
          notifyPlanner: false
          notifyGroup: false
          notifyReviewer: false
        ACTIVATED:
          notifyWorker: true
          notifyPlanner: false
          notifyGroup: false
          notifyReviewer: false
      admin:
        enabled: true
        addresses:
          - "admin1@example.org"
          - "admin2@example.org"
  types:
    TYPE1:
      displayName: Type 1
      quarantine:
        enabled: false
      validation:
        enabled: false
      notification:
        enabled: true
    TYPE2:
      displayName: Type 2
      quarantine:
        enabled: true
      notification:
        enabled: true
        notifyUserOnError: true
        stateConfiguration:
          ALLOCATED:
            notifyReviewer: true
          ACTIVATED:
            notifyPlanner: true
          PREPARED:
            notifyWorker: true
        admin:
          enabled: true
    TYPE3:
      displayName:  Type 3
      workflowType: "SmsOnlineEditingWorkflow"
      geometry:
        growExtents:
          enabled: false
      notification:
        admin:
          enabled: true
          addresses:
            - "admin1@example.org"
            - "admin3@example.org"
            - "admin4@example.org"
```

## Optional Recommended Settings

### Resources and replication settings

Resources and replications settings need to be tuned according to each environment needs and usages.

| Parameter   | Description                                               | Default                                  |
|-------------|-----------------------------------------------------------|------------------------------------------|
| `count`     | The number of replicas to deploy.                         | `1`.                                     |
| `memory`    | The heap space value to make available to each component. | No default value. Example: `1g`.         |
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

### Trusting Additional Certificates

Each 1SMS Application can be configured to trust additional TLS/SSL certificates to support invoking services secured with certificates not signed by a system trusted Certificate Authority. These additional certificates can be provided by an existing Kubernetes Secret. The certificates can be in either the PEM or PKCS12 format. For the PKCS12 format, the files must have `.p12` or `.pfx` or `.pkcs12` extensions and not be password protected. The additional certificates will be trusted in addition to and not instead of the default system trusted certificates.

Note: the entire certificate chain will need to be provided if using a custom Certificate Authority.

| Parameter                    | Description                                                                 | Default |
|------------------------------|-----------------------------------------------------------------------------|---------|
| `trustedCertsExistingSecret` | Existing Kubernetes Secret containing the additional certificates to trust. | `Nil`.  |


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
