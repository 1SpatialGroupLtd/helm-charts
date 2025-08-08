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

**Latest version 1.2.0**

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

## Optional Recommended Settings

### Resources and replication settings

Resources and replications settings need to be tuned according to each environment needs and usages.

|      Parameter      |               Description             | Default                                  |
|---------------------|---------------------------------------|------------------------------------------|
| `count`  | The number of replicas to deploy. | `1`.                                     |
| `memory` | The heap space value to make available to each component. | No default value. Example : `1g`.        |
| `resources`| Configurable [resources][] for the each deployment.| Memory requests and limits both `2048M`. |

_Note: For Production Environments the recommendation is to ensure Guaranteed Quality of Service for each component as per the recommended Kubernetes practices for [QoS][]._

_Note: Recommended number is a minimum of 2 replicas of 1sms-integrate-engine as that would allow for parallel validation sessions. This does depend on the 1Integrate license, please confirm with 1Spatial if you are licensed to run multiple engines._

### Ingress

|      Parameter      |               Description             | Default |
|---------------------|---------------------------------------|---------|
| `ingress.enabled` | Whether 1SMS components needs to be exposed via an ingress. | `true`.|
| `ingress.host`  | The host that will be used to access each 1SMS component on. | ``.|

### TLS

|      Parameter      |               Description             | Default |
|---------------------|---------------------------------------|---------|
| `tls.enabled`   | Whether to deploy with a TLS configuration, port 80 will be exposed if false.| `true`.  |
| `tls.key` | Base64 encoded TLS certificate key, can be for a self-signed certificate. |
| `tls.crt` | Base64 encoded TLS certificate, can be a self-signed certificate.

### Images and container registries

|      Parameter      |               Description             | Default |
|---------------------|---------------------------------------|---------|
| `image.registry` | The registry where the 1SMS docker images can be found, _without_ a trailing slash. | `Nil`.|
| `image.pullPolicy` | [Kubernetes container pull policy][]|`IfNotPresent`.|
| `imagePullSecrets` | [Kubernetes image pull secrets][]| `Nil`.|


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
