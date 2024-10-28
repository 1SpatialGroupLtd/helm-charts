# 1Integrate Chart Settings
By default, 1Integrate will deploy with one fixed user: integrate/integrate1 but this can be changed, see the authentication section below.

## Required Settings

|      Parameter      |               Description             |
|---------------------|---------------------------------------|
| `version` | The docker image tag to use, e.g `4.0.0`. |
| `licence.file` | Base64 encoded 1Integrate licence file. |
| `database.driver` | Database driver to use for the 1Integrate repository, either `oracle`, `sqlserver` or `postgresql`. |
| `database.url` | Full JDBC connection string for the 1Integrate repository. |
| `database.username` | 1Integrate repository database username. |
| `database.password` | 1Integrate repository database password. |

## Optional Settings

| Parameter                          | Description                                                                                                                                                                                                                                                                                      | Default                                         |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------|
| `interface.count`                  | The number of interface replicas to deploy.                                                                                                                                                                                                                                                      | `1`.                                            |
| `interface.memory`                 | The heap space value to make available to engines.                                                                                                                                                                                                                                               | `1g`.                                           |
| `interface.resources`              | Configurable [resources][] for the 1Integrate Interface Deployment.                                                                                                                                                                                                                              | Memory requests and limits both `2048M`.        |
| `interface.opts`                   | Additional system properties to pass into the interfaces.                                                                                                                                                                                                                                        | `Nil`.                                          |
| `interface.affinity`               | Configurable [affinity][] for the 1Integrate Interface Deployment.                                                                                                                                                                                                                               | `{}`.                                           |
| `interface.nodeSelector`           | Configurable [nodeSelector][] for 1Integrate Interface Deployment.                                                                                                                                                                                                                               | `{}`.                                           |
| `interface.tolerations`            | Configurable [tolerations][] for 1Integrate Interface Deployment.                                                                                                                                                                                                                                | `[]`.                                           |
| `interface.priorityClassName`      | The name of the [PriorityClass][].                                                                                                                                                                                                                                                               | `Nil`.                                          |
| `engine.count`                     | The number of engine replicas to deploy.                                                                                                                                                                                                                                                         | `1`.                                            |
| `engine.memory`                    | The heap space value to make available to engines.                                                                                                                                                                                                                                               | `1g`.                                           |
| `engine.resources`                 | Configurable [resources][] for the 1Integrate Engine Deployment.                                                                                                                                                                                                                                 | Memory requests and limits both `2048M`.        |
| `engine.opts`                      | Additional system properties to pass into the engines.                                                                                                                                                                                                                                           | `Nil`.                                          |
| `engine.dynamic`                   | Whether to use dynamic engines or not.  If enabled, the engine count defined above has no effect and engines will be spun up dynamically when sessions start.                                                                                                                                    | `false`.                                        |
| `engine.maxDynamicEngines`         | The upper limit of dynamic engines to be able to start.  Use -1 for unlimited.  If the limit is reached, the session will wait until the number of running engines falls below the limit before starting its engine.                                                                             | `-1`.                                           |
| `engine.affinity`                  | Configurable [affinity][] for the 1Integrate Engine Deployment.                                                                                                                                                                                                                                  | `{}`.                                           |
| `engine.nodeSelector`              | Configurable [nodeSelector][] for 1Integrate Engine Deployment.                                                                                                                                                                                                                                  | `{}`.                                           |
| `engine.tolerations`               | Configurable [tolerations][] for 1Integrate Engine Deployment.                                                                                                                                                                                                                                   | `[]`.                                           |
| `engine.priorityClassName`         | The name of the [PriorityClass][].                                                                                                                                                                                                                                                               | `Nil`.                                          |
| `database.connectionPool.min`      | [4.2+ only] The minimum number of connections to create for the repository database connection pool.                                                                                                                                                                                             | `0`.                                            |
| `database.connectionPool.max`      | [4.2+ only] The maximum number of connections to create for the repository database connection pool. More available connections in the pool will allow 1Integrate to handle more simultaneous user requests. (Ensure your database can handle the given number of simultaneous open connections) | `20`.                                           |
| `database.validation.enabled`      | Whether to validate database connections to the 1Integrate repository, will restore broken connections.                                                                                                                                                                                          | `true`.                                         |
| `database.validation.millis`       | The time in ms between when the database connections to the 1Integrate repository will be validated.                                                                                                                                                                                             | `10000`.                                        |
| `extensions.claimName`                      | The name of an existing PersistentVolumeClaim to mount in as 1Integrate extensions. Allows extentions to be externally stored in external cloud storage such as Azure Blob/File storage or an S3 bucket.                                                                                         | `Nil`.                                           |
| `queueTimeout`                     | The amount of time a session will wait for an engine to become available before erroring.  Use -1 to specify no timeout.                                                                                                                                                                         | `-1`.                                           |
| `image.registry`                   | The registry where the 1Integrate docker images can be found, _without_ a trailing slash.                                                                                                                                                                                                        | `Nil`.                                          |
| `ingress.enabled`                  | Whether 1Integrate needs to be exposed via an ingress.                                                                                                                                                                                                                                           | `true`.                                         |
| `ingress.host`                     | The host that will be used to access 1Integrate on.                                                                                                                                                                                                                                              | `1integrate.local`.                             |
| `image.pullPolicy`                 | [Kubernetes container pull policy](https://kubernetes.io/docs/concepts/containers/images/#updating-images)                                                                                                                                                                                       | `IfNotPresent`.                                 |
| `imagePullSecrets`                 | [Kubernetes image pull secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets)                                                                                                                                                                                | `Nil`.                                          |
| `datastore.destination_file.zip64` | [4.4+ only] Whether to compress all Data Store output files with Zip64, allowing data files larger than 4GB to be created.                                                                                                                                                                       | `false`.                                        |
| `log.level`                        | 1Integrate's logging level, one of `DEBUG`, `INFO`, `WARN`, `ERROR` or `FATAL`                                                                                                                                                                                                                   | `WARN`.                                         |
| `log.cloud`                        | Whether 1Integrate should produce cloud friendly logs. This changes the log format to JSON and enables pod annotations to allow log ingestion by the elastic framework via filebeat                                                                                                              | `false`.                                        |
| `monitoring.enabled`               | Whether monitoring via Elastic APM is enabled                                                                                                                                                                                                                                                    | `false`.                                        |
| `monitoring.endpoint`              | Elastic APM endpoint                                                                                                                                                                                                                                                                             | `Nil`.                                          |
| `monitoring.token`                 | Elastic APM token if required by the above endpoint                                                                                                                                                                                                                                              | `Nil`.                                          |
| `monitoring.environment`           | Elastic APM 'environment' value                                                                                                                                                                                                                                                                  | `Nil`.                                          |
| `monitoring.log.level`             | Elastic APM agent log level                                                                                                                                                                                                                                                                      | `INFO`.                                         |
| `cors.enabled`                     | Whether CORS is enabled                                                                                                                                                                                                                                                                          | `false`.                                        |
| `cors.hosts`                       | The comma separated list of allowed origins for CORS                                                                                                                                                                                                                                             | `false`.                                        |
| `jta.timeout`                      | The JTA timeout value to use in seconds                                                                                                                                                                                                                                                          | `300`.                                          |
| `fileStorage.onDisk`               | Whether to minimise database writes and store uploaded files on disk rather than in the repository. Should not be used with multiple interfaces.                                                                                                                                                 | `false`.                                        |
| `fileStorage.size`                 | Storage size to request for the uploaded files (if enabled).                                                                                                                                                                                                                                     | `1Gi`.                                          |
| `fileStorage.storageClass`         | Storage class to use for the uploaded files (if enabled), this will vary per cloud provider.                                                                                                                                                                                                     | `default`.                                      |
| `cacheStorage.shared`              | Whether to share cache data across all 1Integrate pods, this is required in order for the session data viewer to work.                                                                                                                                                                           | `false`.                                        |
| `cacheStorage.size`                | Storage size to request for the shared cache data (if enabled).                                                                                                                                                                                                                                  | `1Gi`.                                          |
| `cacheStorage.storageClass`        | Storage class to use for the shared cache data (if enabled), this will vary per cloud provider.                                                                                                                                                                                                  | `default`.                                      |
| `tls.enabled`                      | Whether to deploy with a TLS configuration, port 80 will be exposed if false.                                                                                                                                                                                                                    | `true`.                                         |
| `tls.key`                          | Base64 encoded TLS certificate key, can be for a self-signed certificate.                                                                                                                                                                                                                        |
| `tls.crt`                          | Base64 encoded TLS certificate, can be a self-signed certificate.                                                                                                                                                                                                                                |
| `k8s.masterUrl`                    | Kubernetes API endpoint accessible from the interface pods.                                                                                                                                                                                                                                      | `https://kubernetes.default.svc.cluster.local`. |
| `banner`                           | HTML to include as a banner on the login page.  Only the following HTML tags are allowed `<h1> <h2> <h3> <h4> <h5> <p> <span> <br> <text> <body> <title> <html> <head> <meta> <b> <strong> <i> <em> <mark> <small> <del> <ins> <sub> <sup>`                                                      | `Nil`.                                          |
| `office_webhook.url`               | Office 365 webhook URL for delivering session state notifications to applications such as Microsoft Teams.                                                                                                                                                                                       | `Nil`.                                          |
| `office_webhook.onError`           | Whether to notify the webhook that a session has errored.  Includes error details                                                                                                                                                                                                                | `false`.                                        |
| `office_webhook.onPause`           | Whether to notify the webhook that a session has paused.                                                                                                                                                                                                                                         | `false`.                                        |
| `office_webhook.onRun`             | Whether to notify the webhook that a session has started running.                                                                                                                                                                                                                                | `false`.                                        |
| `office_webhook.onFinish`          | Whether to notify the webhook that a session has finished.                                                                                                                                                                                                                                       | `false`.                                        |

## Authentication
### Fixed Users
By default, 1Integrate deploys in fixed user mode.  Custom users can be created by setting the `authentication.users` parameter. 
> This is not recommended for production.

For example:
```yaml
authentication:
  users:
    newuser1: password123
    newuser2: password123
    newuser3: password123
```

### SAML
1Integrate can also be configured to use SAML for User Interface authentication.
> SAML support requires 1Integrate 5.0.0+ and helm chart 1.3.0+

For example:
```yaml
authentication:
  saml:
    enabled: true
    entityId: 1integrate
    keyStore: bXliYXNlNjRlbmNvZGVkc2FtbGtlc3RvcmU
    keyStorePassword: mykeystorepassword
    keyAlias: mykeyalias
    keyPassword: mykeypassword
    certificateAlias: mycertificatealias
    principalAttribute: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
    roleAttribute: http://schemas.microsoft.com/ws/2008/06/identity/claims/role
    metadataUrl: https://mysamlprovidermetadataurl
    loginUrl: https://mysamlproviderloginurl
    logoutUrl: https://mysamlproviderlogouturl
```

### OpenID Connect
1Integrate can also be configured to use OpenID Connect for User Interface authentication.
> OpenID Connect support requires 1Integrate 4.2.0+ and helm chart 1.1.0+

For example:
```yaml
authentication:
  oidc:
    enabled: true
    providerUrl: https://myoidcprovider
    sslRequired: all
```

### LDAP
Additionally, 1Integrate can be configured to use LDAP for authentication (and optionally authorisation).  Enable this by setting `authentication.ldap.enabled` to `true` and by providing your LDAP config as part
of the `authentication.ldap.config` parameter.  The values needed here are the [standard 1Integrate LDAP parameters](https://1spatial.com/documentation/Topics/Installation/LDAP_ForCloud.pdf) albeit formatted for YAML.  User access can be controlled through LDAP groups or standard application role mapping as covered below.

For example:
```yaml
authentication:
  ldap:
    enabled: true
    config:
      ldap.host: <ldap_host>
      ldap.principal: <ldap_bind_principal>
      ldap.credential: <ldap_bind_credential>
      ldap.user.base.dn: <ldap_user_base_dn>
      ldap.group.base.dn: <ldap_group_base_dn>
```

## Authorisation
Some authentication mechanism require application roles to be mapped to users to allow them to use the product, e.g. LDAP.  This is controlled by the `authentication.roleMapping` parameter.  A breakdown of the application roles can be found [as part of the product documentation](https://1spatial.com/documentation/Topics/Installation/LDAP_ForCloud.pdf).
For fixed users, the roles are assigned to each username.  For LDAP, the roles are assigned to LDAP groups.

For example:
```yaml
integrate:
  authentication:
    roleMapping:
      newuser1: 1int-admin
      newuser2: 1int-user
      newuser3: 1int-user
```

## Full Config Examples


### SAML
> Requires 1Integrate 5.0.0+
```yaml
version: 5.0.0
licence:
  file: aWFtYXNlY3JldGxpY2VuY2VmaWxl
database:
  driver: postgresql
  url: jdbc:oracle:thin:@postgres/postgres
  username: postgres
  password: postgres
authentication:
  saml:
    enabled: true
    entityId: 1integrate
    keyStore: bXliYXNlNjRlbmNvZGVkc2FtbGtlc3RvcmU
    keyStorePassword: mykeystorepassword
    keyAlias: mykeyalias
    keyPassword: mykeypassword
    certificateAlias: mycertificatealias
    principalAttribute: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
    roleAttribute: http://schemas.microsoft.com/ws/2008/06/identity/claims/role
    metadataUrl: https://mysamlprovidermetadataurl
    loginUrl: https://mysamlproviderloginurl
    logoutUrl: https://mysamlproviderlogouturl
interface:
  count: 1
engine:
  count: 2
cacheStorage:
  shared: true
tls:
  key: aWFtYXNlY3JldHRsc2tleQ
  crt: aWFtYXNlY3JldHRsc2NlcnRpZmljYXRl
image:
  registry: cloudcr.io
ingress:
  host: 1integrate.cloud.com
```

### OpenID Connect
```yaml
version: 4.2.0
licence:
  file: aWFtYXNlY3JldGxpY2VuY2VmaWxl
database:
  driver: postgresql
  url: jdbc:oracle:thin:@postgres/postgres
  username: postgres
  password: postgres
authentication:
  oidc:
    enabled: true
    providerUrl: https://myoidcprovider
    sslRequired: all
interface:
  count: 1
engine:
  count: 2
cacheStorage:
  shared: true
tls:
  key: aWFtYXNlY3JldHRsc2tleQ
  crt: aWFtYXNlY3JldHRsc2NlcnRpZmljYXRl
image:
  registry: cloudcr.io
ingress:
  host: 1integrate.cloud.com
```

### LDAP
```yaml
version: 4.0.0
licence:
  file: aWFtYXNlY3JldGxpY2VuY2VmaWxl
database:
  driver: postgresql
  url: jdbc:oracle:thin:@postgres/postgres
  username: postgres
  password: postgres
authentication:
  ldap:
    enabled: true
    config:
      ldap.host: dc.local
      ldap.principal: user1
      ldap.credential: password123
      ldap.user.base.dn: ou=users,dc=local
      ldap.group.base.dn: cn=users,dc=local
  roleMapping:
    group1: 1int-admin
interface:
  count: 1
engine:
  count: 2
cacheStorage:
  shared: true
tls:
  key: aWFtYXNlY3JldHRsc2tleQ
  crt: aWFtYXNlY3JldHRsc2NlcnRpZmljYXRl
image:
  registry: cloudcr.io
ingress:
  host: 1integrate.cloud.com
```

### Fixed Users
> Not recommended for production
```yaml
integrate:
version: 4.0.0
licence:
  file: aWFtYXNlY3JldGxpY2VuY2VmaWxl
database:
  driver: postgresql
  url: jdbc:oracle:thin:@postgres/postgres
  username: postgres
  password: postgres
authentication:
  users:
    newuser1: password123
    newuser2: password123
    newuser3: password123
  roleMapping:
    newuser1: 1int-admin
    newuser2: 1int-user
    newuser3: 1int-user
interface:
  count: 1
engine:
  count: 2
cacheStorage:
  shared: true
tls:
  key: aWFtYXNlY3JldHRsc2tleQ
  crt: aWFtYXNlY3JldHRsc2NlcnRpZmljYXRl
image:
  registry: cloudcr.io
ingress:
  host: 1integrate.cloud.com
```

Copyright (c) 1Spatial 2024


[affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
[nodeSelector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
[priorityClass]: https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#priorityclass
[tolerations]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[resources]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container
