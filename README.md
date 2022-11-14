# Official 1Spatial Helm Charts

This repository contains [Helm](https://helm.sh) charts for 1Spatial products.

## Available Charts

* [1Integrate](https://1spatial.com/products/1integrate/)

## Usage

To add the 1Spatial charts repository:  
`helm repo add 1spatial https://1spatialgroupltd.github.io/helm-charts/`

## Installing

It's recommended (but not required) that our product charts are installed in their own namespaces to logically separate product configurations.

While the charts do contain a number of default parameters, the charts require a number of parameters to be provided for install.  These can be provided as part of the command line or defined in a yaml file passed into the `install` command.  See below for an overview of the required parameters, optional parameters and full configuration examples for each product.

`helm install <deployment_name> 1spatial/<chart_name> -n <namespace> -f <settings_file>.yaml`

e.g. for 1Integrate
`helm install 1integrate 1spatial/1integrate -n 1integrate -f 1integrate.yaml`

### 1Integrate Settings
By default, 1Integrate will deploy with one fixed user: integrate/integrate1 but this can be changed, see the authentication section below.

#### Required Settings

|      Parameter      |               Description             |
|---------------------|---------------------------------------|
| `integrate.version` | The docker image tag to use, e.g `3.0.0`. |
| `integrate.licence.file` | Base64 encoded 1Integrate licence file. |
| `integrate.database.driver` | Database driver to use for the 1Integrate repository, either `oracle`, `sqlserver` or `postgresql`. |
| `integrate.database.url` | Full JDBC connection string for the 1Integrate repository. |
| `integrate.database.username` | 1Integrate repository database username. |
| `integrate.database.password` | 1Integrate repository database password. |
| `integrate.tls.key` | Base64 encoded TLS certificate key, can be for a self-signed certificate. |
| `integrate.tls.crt` | Base64 encoded TLS certificate, can be a self-signed certificate. |

#### Optional Settings

|      Parameter      |               Description             |                    Default                |
|---------------------|---------------------------------------|-------------------------------------------|
| `integrate.interface.count` | The number of interface replicas to deploy. |  `1`. |
| `integrate.interface.memory` | The memory in megabytes to make available to interfaces.  This number will be the memory actually available to the application.  The kubernetes memory will be determined from this: requested memory will be +256MB while the overall limit will be +1GB. |  `1024`. |
| `integrate.interface.opts` | Additional system properties to pass into the interfaces. |  `Nil`. |
| `integrate.engine.count` | The number of engine replicas to deploy. |  `1`. |
| `integrate.engine.memory` | The memory in megabytes to make available to engines.  This number will be the memory actually available to the application.  The kubernetes memory will be determined from this: requested memory will be +256MB while the overall limit will be +1GB. |  `1024`. |
| `integrate.engine.opts` | Additional system properties to pass into the engines. |  `Nil`. |
| `integrate.engine.dynamic` | Whether to use dynamic engines or not.  If enabled, the engine count defined above has no effect and engines will be spun up dynamically when sessions start. |  `false`. |
| `integrate.engine.maxDynamicEngines` | The upper limit of dynamic engines to be able to start.  Use -1 for unlimited.  If the limit is reached, the session will wait until the number of running engines falls below the limit before starting its engine. |  `-1`. |
| `integrate.queueTimeout` | The amount of time a session will wait for an engine to become available before erroring.  Use -1 to specify no timeout. |  `-1`. |
| `integrate.image.registry` | The registry where the 1Integrate docker images can be found, _without_ a trailing slash. |  `Nil`. |
| `integrate.ingress.host` | The host that will be used to access 1Integrate on. |  `1integrate.local`. |
| `integrate.image.pullPolicy` | [Kubernetes container pull policy](https://kubernetes.io/docs/concepts/containers/images/#updating-images) |  `IfNotPresent`. |
| `integrate.imagePullSecrets` | [Kubernetes image pull secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets) |  `Nil`. |
| `integrate.log.level` | 1Integrate's logging level, one of `DEBUG`, `INFO`, `WARN`, `ERROR` or `FATAL` |  `WARN`. |
| `integrate.log.cloud` | Whether 1Integrate should produce cloud friendly logs. This changes the log format to JSON and enables pod annotations to allow log ingestion by the elastic framework via filebeat |  `false`. |
| `integrate.monitoring.enabled` | Whether monitoring via Elastic APM is enabled |  `false`. |
| `integrate.monitoring.endpoint` | Elastic APM endpoint |  `Nil`. |
| `integrate.monitoring.token` | Elastic APM token if required by the above endpoint |  `Nil`. |
| `integrate.monitoring.environment` | Elastic APM 'environment' value |  `Nil`. |
| `integrate.monitoring.log.level` | Elastic APM agent log level |  `INFO`. |
| `integrate.cors.enabled` | Whether CORS is enabled |  `false`. |
| `integrate.cors.allowed.origins` | The list of allowed origins for CORS |  `false`. |
| `integrate.jta.timeout` | The JTA timeout value to use in seconds |  `300`. |
| `integrate.fileStorage.onDisk` | Whether to minimise database writes and store uploaded files on disk rather than in the repository. |  `false`. |
| `integrate.fileStorage.size` | Storage size to request for the uploaded files (if enabled). |  `1Gi`. |
| `integrate.fileStorage.storageClass` | Storage class to use for the uploaded files (if enabled), this will vary per cloud provider. |  `default`. |
| `integrate.cacheStorage.shared` | Whether to share cache data across all 1Integrate pods, this is required in order for the session data viewer to work. |  `false`. |
| `integrate.cacheStorage.size` | Storage size to request for the shared cache data (if enabled). |  `1Gi`. |
| `integrate.cacheStorage.storageClass` | Storage class to use for the shared cache data (if enabled), this will vary per cloud provider. |  `default`. |
| `integrate.k8s.masterUrl` | Kubernetes API endpoint accessible from the interface pods. |  `https://kubernetes.default.svc.cluster.local`. |
| `integrate.banner` | HTML to include as a banner on the login page.  Only the following HTML tags are allowed `<h1> <h2> <h3> <h4> <h5> <p> <span> <br> <text> <body> <title> <html> <head> <meta> <b> <strong> <i> <em> <mark> <small> <del> <ins> <sub> <sup>`  |  `Nil`. |
| `integrate.office_webhook.url` | Office 365 webhook URL for delivering session state notifications to applications such as Microsoft Teams. |  `Nil`. |
| `integrate.office_webhook.onError` | Whether to notify the webhook that a session has errored.  Includes error details |  `false`. |
| `integrate.office_webhook.onPause` | Whether to notify the webhook that a session has paused. |  `false`. |
| `integrate.office_webhook.onRun` | Whether to notify the webhook that a session has started running. |  `false`. |
| `integrate.office_webhook.onFinish` | Whether to notify the webhook that a session has finished. |  `false`. |

#### Authentication
##### Fixed Users
By default, 1Integrate deploys with a fixed user, integrate.  Custom users can be created by overriding the `integrate.authentication.users` parameter.

For example:
```yaml
integrate:
  authentication:
    users:
      newuser1: password123
      newuser2: password123
      newuser3: password123
```

##### LDAP
Additionally, 1Integrate can be configured to use LDAP for authentication (and optionally authorisation).  Enable this by setting `integrate.authentication.ldap.enabled` to `true` and by providing your LDAP config as part
of the `integrate.authentication.ldap.config` parameter.  The values needed here are the [standard 1Integrate LDAP parameters](https://1spatial.com/documentation/Topics/Installation/LDAP_ForCloud.htm) albeit formatted for YAML.  User access can be controlled through LDAP groups or standard application role mapping as covered below.

For example:
```yaml
integrate:
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

#### Authorisation
However the users have been defined, explicitly or from LDAP, application roles need to be mapped to allow them to use the product.  This is controlled by the `integrate.authentication.roleMapping` parameter.  A breakdown of the application roles can be found [as part of the product documentation](https://1spatial.com/documentation/Topics/Installation/LDAP_ForCloud.htm).
For fixed users, the roles are assigned to each username.  For LDAP, the roles are assigned to LDAP groups.

For example:
```yaml
integrate:
  authentication:
    roleMapping:
      newuser1: rs_admins,rs_users,rswsuser
      newuser2: rs_users,rswsuser
      newuser3: rs_users,rswsuser
```

#### Full Config Examples
##### Fixed Users
```yaml
integrate:
  version: 3.0.0
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
      newuser1: rs_admins,rs_users,rswsuser
      newuser2: rs_users,rswsuser
      newuser3: rs_users,rswsuser
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

##### LDAP
```yaml
integrate:
  version: 3.0.0
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
      group1: rs_admins,rs_users,rswsuser
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

Copyright (c) 1Spatial 2021
