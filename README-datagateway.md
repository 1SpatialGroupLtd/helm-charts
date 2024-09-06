# 1Data Gateway Helm Chart

This Helm chart deploys 1Data Gateway into Kubernetes.
It does not deploy either of it's required services (PostgreSQL and 1Integrate).
Chart parameters are provided to allow you to configure the location of these dependencies.

To guarantee all documented configuration values are supported use Helm chart version 0.6.0 with a 1Data Gateway image of version 2.14.1 or greater.

## Required Parameters

* `datagateway.db.jdbcUrl`: JDBC URL for the 1DG configuration database.
    While this chart has only been tested with PostgreSQL, any database supported by 1DG should work.
* `datagateway.db.username`: Username for the 1DG configuration database (if required).
* `datagateway.db.password`: Password for the 1DG configuration database (if required).


* `ingress.hosts` : List of `host: <domain name>` pairs to route requests to 1DG.


* `image.tag`: The image tag (1DataGateway version) to deploy.

## Commonly-used Optional Parameters

### JVM Configuration

* `datagateway.memory`: The maximum amount of memory allocated to the Java Heap.
    If unconfigured then the system default will be used.
    Example allowed values: "512m", "1g" (default unconfigured).
* `datagateway.opts`: Additional JVM options to pass into the service (default "").

### Media Persistent Volume Claim Configuration

* `datagateway.mediaPvc.storageClass`: Sets the storage class used to allocate PV storage to the 1DG media PV claim (defaults to 'default').
* `datagateway.mediaPvc.size`: Sets the media PVC size (default 1GB).
* `datagateway.mediaPvc.volumeName`: Specify the name of a (pre-existing) PV to use (default "" to auto-provision).

### Database Configuration

* `datagateway.db.maxConnections`: Maximum number of connections in the database connection pool.
    Be careful overriding this property - more connections are not always better.
    More information is available in the guide: [About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing).
    If empty defaults to HikariCP default 10 connections (default "").

### Virus Scanner Configuration <sup>1</sup>

* `datagateway.clamd.host`: Hostname to use for clamd virus scanner (default "").
    If set to an empty string, then the virus scanner will be disabled.
    The clamd host should have the 1DG media directory mounted at the same location as the 1DG container (default: /1datagateway/media).
* `datagateway.clamd.port`: Port to connect to for virus scanning (default 3310).

### 1Integrate Connection Settings

* `datagateway.integrate.baseUri`: Location of the 1Integrate rest api (default "http://localhost:8080/1Integrate/rest/"). <sup>1</sup>
* `datagateway.integrate.username`: Username for 1DG to authenticate with 1Integrate (default ""). <sup>1</sup>
* `datagateway.integrate.password`: Password for 1DG to authenticate with 1Integrate (default ""). <sup>1</sup>
* `datagateway.integrate.apiKey`: API Key for 1DG to authenticate with 1Integrate. <sup>1</sup>
    If property is not empty then authentication mode will be set to use API Key. (default ""). <sup>1</sup>
* `datagateway.integrate.cleanupInterval`: The interval at which 1Integrate sessions for finished submissions will be removed.
    Must be a valid duration (default 5m).

### Mail Sender Settings <sup>1</sup>

* `datagateway.mail.enabled`: If true, enable the mail sender. (default false).
* `datagateway.mail.address`: Address to send mail from.
    Must be a valid email address according to [RFC822](https://www.ietf.org/rfc/rfc822.txt). (default unconfigured).
* `datagateway.mail.protocol`: Mail protocol, either "SMTP" or "SMTPS". (default unconfigured).
* `datagateway.mail.host`: Mail server host name. (default unconfigured).
* `datagateway.mail.port`: Mail server port. (default unconfigured).
* `datagateway.mail.security`: Security protocol, either "NONE", "TLS_OPTIONAL" or "TLS". (default unconfigured).
* `datagateway.mail.username`: Username for 1DG to authenticate with mail sender (default unconfigured).
* `datagateway.mail.password`: Password for 1DG to authenticate with mail sender (default unconfigured).

### Site Settings <sup>1</sup>

* `datagateway.site.name`: The name of the site (default unconfigured).
* `datagateway.site.rootUrl`: The base URL where the site is accessed (if deployed behind a proxy this should be the publicly accessible address).
    Must be a valid URI specifying the scheme and hostname with no user info, query or fragment (default unconfigured).
* `datagateway.site.loginTitle`: The title displayed to the unauthenticated user on the login screen (default unconfigured).
* `datagateway.site.helpUrl`: Overrides the default help URL.
    Must be a valid URI specifying the scheme and hostname with no user info, query or fragment (default unconfigured).

### APM Monitoring Configuration

* `datagateway.apm.enabled`: If true, enable the APM (Application Performance Management) Java Agent (default false).
* `datagateway.apm.serviceName`: [Service name](https://www.elastic.co/guide/en/apm/agent/java/current/config-core.html#config-service-name) (default unconfigured).
* `datagateway.apm.serverUrl`: [APM server URL](https://www.elastic.co/guide/en/apm/agent/java/current/config-reporter.html#config-server-url) (default '1datagateway').
* `datagateway.apm.applicationPackages`: [Application packages](https://www.elastic.co/guide/en/apm/agent/java/current/config-stacktrace.html#config-application-packages) used to determine which code in-app vs library (default 'com.onespatial.*').
* `datagateway.apm.secretToken`: [APM secret token](https://www.elastic.co/guide/en/apm/agent/java/current/config-reporter.html#config-secret-token) (default unconfigured).
* `datagateway.apm.environment`: [Environment name](https://www.elastic.co/guide/en/apm/agent/java/current/config-core.html#config-environment) (default unconfigured).
* `datagateway.apm.enableExperimentalInstrumentations`: [Enable Experimental Instrumentations](https://www.elastic.co/guide/en/apm/agent/java/current/config-core.html#config-enable-experimental-instrumentations) (default false).
* `datagateway.apm.logLevel`: [APM Agent Log Level](https://www.elastic.co/guide/en/apm/agent/java/current/config-logging.html#config-log-level) (default 'INFO')

### Actuator Configuration

* `datagateway.actuator.enabled`: If true enable `/actuator` endpoints (default false).
* `datagateway.actuator.expose`: List of actuator endpoints to expose.
    See [Spring Boot Documentation](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html) for all options, for example: startup, configprops, env, flyway, httpexchanges. (default: health).
* `datagateway.actuator.healthDetails`: Whether to include detailed information in health response.
    Possible values: never, always and when_authorized (when_authorized will only show details to authorized users for 1DG version 3.0.0 or greater). (default: when_authorized).
* `datagateway.actuator.port`: The port to expose the actuator endpoints on.
    If different from service port then not exposed outside the pod by default but could be used for liveness/readiness probes. (default: same as service).

Note: For production deployments it is advisable to create a mechanism for alerting operational support if the service becomes unhealthy.
One way this can be achieved is to set `datagateway.actuator.enabled` to true and then periodically call `/actuator/health`, alerting support if the response code is not 200, so they can investigate the cause of the issue.

### Liveness / Readiness Checks

* `livenessProbe`: Liveness probe configuration - should the instance be replaced? (default: HTTP request to `/`).
* `readinessProbe`: Readiness probe configuration - should the instance serve traffic? (default: HTTP request to `/api/v1/application`).

For liveness and readiness probe config we support [all config provided by kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).
Add required keys configuration below the top level value keys.

Note: If actuator endpoints are enabled above then the liveness and readiness paths can be updated to `/actuator/health/liveness` and `/actuator/health/readiness` endpoints respectively.
When using the acutator endpoints with `datagateway.actutor.port` set remember to update the port used by your probes to match.

### Ingress Configuration

* `ingress.enabled`: If true, enable a default ingress resource (default true).
* `ingress.className`: Links ingress resource to an ingress controller (default nginx).
* `ingress.annotations`: Additional annotations to add to the ingress resource.

### TLS

* `tls.enabled`: If true, enable TLS on the 1DG HTTP service (default false).
* `tls.secretName`: Name of a k8s secret of type kubernetes.io/tls, used for the TLS-enabled ingress.

### Image Configuration

* `imagePullSecrets`: Array of secrets used for pulling Docker images.
* `image.registry`: The registry where the 1DG docker images can be found, without a trailing slash (default "").
* `image.repository`: Repository from which to pull Docker images.
* `image.pullPolicy`: Allows overriding the default pull policy of IfNotPresent.

### Miscellaneous

* `nameOverride`: Allows overriding the chart name as used in the various chart resources.
* `fullnameOverride`: Allows overriding the fully-qualified app name.

* `datagateway.config`: Workaround while required properties have not yet been added to the helm chart. Allows arbitrary YAML to be added to the application config. (default {}).

* `datagateway.defaultAdminPasswordSecret`: The name of the secret storing the default admin password.
    Admin password will be updated to this value when the server restarts.
    This can be created for example like `kubectl create secret generic mysecret --from-literal=password='password'`.
    Note: if you need to change the password on upgrade then create a new secret and update your values file since the server needs to restart to pick up the new password and this is not triggered by a change of the secret value. (default undefined).

## Annotations
*  <sup>1</sup> These values are only applied during initial chart installation.
They will be ignored on update if the database has already been populated with data.