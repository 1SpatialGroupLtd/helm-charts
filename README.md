# Official 1Spatial Helm Charts

This repository contains [Helm](https://helm.sh) charts for 1Spatial products, used for deploying into Kubernetes environments. They are provided to help administrators install 1Spatial products by adding the helm repository to their environment and setting environment-specific values.

## Available Charts

* [1Integrate](https://1spatial.com/products/1integrate/)
* [1Data Gateway](https://1spatial.com/products/1data-gateway/)
* [1Spatial Management Suite](https://1spatial.com/products/other-products/1spatial-management-suite/)

## Usage

To add the 1Spatial charts repository:  
`helm repo add 1spatial https://1spatialgroupltd.github.io/helm-charts/`

## Installing

It's recommended (but not required) that our product charts are installed in their own namespaces to logically separate product configurations.

While the charts do contain a number of default parameters, the charts require a number of parameters to be provided for install.  These can be provided as part of the command line or defined in a yaml file passed into the `install` command.  See below for an overview of the required parameters, optional parameters and full configuration examples for each product.

`helm install <deployment_name> 1spatial/<chart_name> -n <namespace> -f <settings_file>.yaml`

e.g. for 1Integrate
`helm install 1integrate 1spatial/1integrate -n 1integrate -f 1integrate.yaml`

### Chart Settings

* [1Integrate](README-1integrate.md)
* [1Data Gateway](README-datagateway.md)
* [1Spatial Management Suite Gateway](README-1sms.md)

Copyright (c) 1Spatial 2021
