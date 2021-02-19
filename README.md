# Google Cloud Platform (GCP) Reference Platform

This reference platform `Configuration` for Kubernetes and Data Services is a starting point to
build, run, and operate your own internal cloud platform and offer a self-service console and API to
your internal teams.

It provides platform APIs to provision fully configured GKE clusters, with secure networking, and
stateful cloud services (Cloud SQL) designed to securely connect to the nodes in each GKE cluster --
all composed using cloud service primitives from the [Crossplane GCP
Provider](https://doc.crds.dev/github.com/crossplane/provider-gcp). App deployments can securely
connect to the infrastructure they need using secrets distributed directly to the app namespace.

## Requirements

A fully working Kubernetes cluster with `kubectl` configured for accessing to it.

Crossplane fully installed on it, you can refer to the [official documentation](https://crossplane.io/docs/v1.0/getting-started/install-configure.html) for installing method.

In addition to Crossplane the GCP and Helm provider must be installed, follow the [official guide](https://crossplane.io/docs/v1.0/introduction/providers.html).

#### Install the Crossplane kubectl extension (for convenience)

```console
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
cp kubectl-crossplane /usr/local/bin
```

#### Install the Platform Configuration

```console
PLATFORM_CONFIG=regitry.io/platform-ref-gcp:latest

kubectl crossplane install configuration ${PLATFORM_CONFIG}
kubectl get pkg
```
#### GCP Provider Setup

Set up your GCP account keyfile by following the instructions on:
https://crossplane.io/docs/v1.0/getting-started/install-configure.html#select-provider

Ensure that the following roles are added to your service account:

* `roles/compute.networkAdmin`
* `roles/container.admin`
* `roles/iam.serviceAccountUser`

Then create the secret using the given `creds.json` file:

```console
kubectl create secret generic gcp-creds -n crossplane-system --from-file=key=./creds.json
```

Create the `ProviderConfig`, ensuring to set the `projectID` to your specific GCP project:

```console
kubectl apply -f examples/provider-default-gcp.yaml
```
#### App Dev/Ops: Consume the infrastructure you need using kubectl

Now you can provide with kubectl the infrastructure in the examples folder:

```console
kubectl apply -f examples/<file-name.yaml>
```

1. network: 
2. cluster:
3. sql:

### Cleanup & Uninstall
#### Cleanup Resources

* Using `kubectl delete -f examples/<file-name.yaml>`.

Verify all underlying resources have been cleanly deleted:

```console
kubectl get managed
```

#### Uninstall Provider & Platform Configuration

```console
kubectl delete configurations.pkg.crossplane.io platform-ref-gcp
kubectl delete providers.pkg.crossplane.io provider-gcp
kubectl delete providers.pkg.crossplane.io provider-helm
```

## APIs in this Configuration

* `Cluster` - provision a fully configured Kubernetes cluster
  * [definition.yaml](cluster/definition.yaml)
  * [composition.yaml](cluster/composition.yaml) includes (transitively):
    * `GKECluster`
    * `NodePool`
    * `HelmReleases` for Prometheus and other cluster services.
* `Network` - fabric for a `Cluster` to securely connect the control plane, pods, and services
  * [definition.yaml](network/definition.yaml)
  * [composition.yaml](network/composition.yaml) includes:
    * `Network`
    * `Subnetwork`

## Customize 

Clone the GitHub repo.

```console
git clone https://github.com/upbound/platform-ref-gcp.git
cd platform-ref-gcp
```

Login to your container registry.

```console
docker login ${REGISTRY} -u ${USER_LOGIN}
```

Build package.

```console
kubectl crossplane build configuration --name package.xpkg --ignore "examples/*"
```

Push package to registry.

```console
kubectl crossplane push configuration ${PLATFORM_CONFIG} -f package.xpkg
```

Install package into an Upbound `Platform` instance.

```console
kubectl crossplane install configuration ${PLATFORM_CONFIG}
```

The cloud service primitives that can be used in a `Composition` today are
listed in the Crossplane provider docs:

* [Crossplane GCP Provider](https://doc.crds.dev/github.com/crossplane/provider-gcp)

To learn more see [Configuration
Packages](https://crossplane.io/docs/v0.14/getting-started/package-infrastructure.html).

## Learn More

If you're interested in building your own reference platform for your company,
we'd love to hear from you and chat. You can setup some time with us at
info@upbound.io.

For Crossplane questions, drop by [slack.crossplane.io](https://slack.crossplane.io), and say hi!

