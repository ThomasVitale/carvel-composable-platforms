# Platforms with Carvel

This guide explains how to deploy a platform built with the Carvel tools on Kubernetes.

## Prerequisites

Ensure you have the following tools installed in your local environment:

* [Podman Desktop](https://www.thomasvitale.com/podman-desktop-for-java-development) (or Docker)
* Carvel [`kctrl`](https://carvel.dev/kapp-controller/docs/latest/install/#installing-kapp-controller-cli-kctrl)
* Carvel [`kapp`](https://carvel.dev/kapp/docs/latest/install)
* [kind](https://kind.sigs.k8s.io).

Then, create a local Kubernetes cluster with `kind`.

```shell script
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kadras
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Deploy Carvel kapp-controller

The platform relies on the Kubernetes-native package management capabilities offered by Carvel kapp-controller. You can install it with Carvel `kapp` (recommended choice) or `kubectl`.

```shell script
kapp deploy -a kapp-controller -y \
  -f https://github.com/carvel-dev/kapp-controller/releases/latest/download/release.yml
```

## Add the Kadras Repository

Add the Kadras repository to make the platform packages available to the cluster.

```shell script
kctrl package repository add -r kadras-packages \
  --url ghcr.io/kadras-io/kadras-packages:0.14.2 \
  -n kadras-packages --create-namespace
```

## Configure the Platform

The installation of the Kadras platform can be configured via YAML. Create a `values.yml` file with any configuration you need for the platform. The following is a minimal configuration example for a local installation.

```yaml
platform:
  profile: run
  infrastructure_provider: local
  ingress:
    domain: 127.0.0.1.sslip.io
```

## Install the Platform

Reference the `values.yml` file you created in the previous step and install the Kadras Engineering Platform.

```shell script
kctrl package install -i engineering-platform \
  -p engineering-platform.packages.kadras.io \
  -v 0.12.2 \
  -n kadras-packages \
  --values-file values.yml
```

## Verify the Installation

Verify that all the platform components have been installed and properly reconciled.

```shell script
kctrl package installed list -n kadras-packages
```

## Use the Platform

You can now use the platform to deploy your containerized applications using the Knative API via a manifest or using the [kn](https://knative.dev/docs/getting-started/quickstart-install/#install-the-knative-cli) CLI.

```shell script
kn service create hello --image ghcr.io/knative/helloworld-go
```

The application will be available at http://hello.default.127.0.0.1.sslip.io.

## Cleanup

When you're done working with the platform, you can delete the local cluster as follows.

```shell script
kind delete cluster --name kadras
```
