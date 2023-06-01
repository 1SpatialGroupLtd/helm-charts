# 1Data Gateway Helm Chart

This Helm chart deploys 1Data Gateway into Kubernetes.
It does not deploy either of it's required services (PostgreSQL and 1Integrate).
Chart parameters are provided to allow you to configure the location of these dependencies.

## Required Parameters

* `datagateway.db.jdbcUrl`: JDBC URL for the 1DG configuration database.
  While this chart has only been tested with PostgreSQL, any database supported by 1DG should work.
* `datagateway.db.username`: Username for the 1DG configuration database (if required).
* `datagateway.db.password`: Password for the 1DG configuration database (if required).


* `ingress.hosts` : List of `host: <domain name>` pairs to route requests to 1DG.

## Commonly-used Optional Parameters

* `imagePullSecrets`: Array of secrets used for pulling Docker images.


* `datagateway.mediaPvc.storageClass`: Sets the storage class used to allocate PV storage to the 1DG media PV claim (defaults to 'default').
* `datagateway.mediaPvc.size`: Sets the media PVC size (default 1GB).
* `datagateway.mediaPvc.volumeName`: Specify the name of a (pre-existing) PV to use (default "" to auto-provision).


* `datagateway.clamd.host`: Hostname to use for clamd virus scanner (default "").
  If set to an empty string, then the virus scanner will be disabled.
  The clamd host should have the 1DG media directory mounted at the same location as the 1DG container (default: /1datagateway/media).
* `datagateway.clamd.port`: Port to connect to for virus scanning (default 3310).


* `datagateway.integrate.baseUri`: Location of the 1Integrate rest api (default "http://localhost:8080/1Integrate/rest/").
* `datagateway.integrate.username`: Username for 1DG to authenticate with 1Integrate (default "").
* `datagateway.integrate.password`: Password for 1DG to authenticate with 1Integrate (default "").


* `ingress.enabled`: If true, enable a default ingress resource (default true).
* `ingress.className`: Links ingress resource to an ingress controller (default nginx).


* `tls.enabled`: If true, enable TLS on the 1DG HTTP service (default false).
* `tls.secretName`: Name of a k8s secret of type kubernetes.io/tls, used for the TLS-enabled ingress.


* `image.registry`: The registry where the 1DG docker images can be found, without a trailing slash (default "").
* `image.repository`: Repository from which to pull Docker images.
* `image.pullPolicy`: Allows overriding the default pull policy of IfNotPresent.
* `image.tag`: Overrides the image tag whose default is the chart appVersion.


* `nameOverride`: Allows overriding the chart name as used in the various chart resources.
* `fullnameOverride`: Allows overriding the fully-qualified app name.
