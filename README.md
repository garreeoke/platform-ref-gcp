# Google Cloud Platform (GCP) Reference Platform

This reference platform `Configuration` for Kubernetes and Data Services is a starting point to
build, run, and operate your own internal cloud platform and offer a self-service console and API to
your internal teams.

It provides platform APIs to provision fully configured GKE clusters, with secure networking, and
stateful cloud services (Cloud SQL) designed to securely connect to the nodes in each GKE cluster --
all composed using cloud service primitives from the [Crossplane GCP
Provider](https://doc.crds.dev/github.com/crossplane/provider-gcp). App deployments can securely
connect to the infrastructure they need using secrets distributed directly to the app namespace.

# Installing UXP on a Kubernetes Cluster

The other option is installing UXP into a Kubernetes cluster you manage using `up`, which
is the official CLI for interacting with Upbound Cloud and Universal Crossplane (UXP).

There are multiple ways to [install up](https://cloud.upbound.io/docs/cli/#install-script),
including Homebrew and Linux packages.

```console
curl -sL https://cli.upbound.io | sh
```

Ensure that your kubectl context is pointing to the correct cluster:

```console
kubectl config current-context
```

Install UXP into the `upbound-system` namespace:

```console
up uxp install
```

Validate the install using the following command:

```console
kubectl -n upbound-system get all
```

#### Install the Crossplane kubectl extension (for convenience)

Now that your kubectl context is configured to connect to a UXP Control Plane,
we can install this reference platform as a Crossplane package.

```console
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
cp kubectl-crossplane /usr/local/bin
```

#### Install the Platform Configuration

```console
PLATFORM_VERSION=v0.0.7
PLATFORM_CONFIG=registry.upbound.io/upbound/platform-ref-gcp:${PLATFORM_VERSION}

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
kubectl -n upbound-system create secret generic gcp-creds --from-file=key=./creds.json
```

Create the `ProviderConfig`, ensuring to set the `projectID` to your specific GCP project:

```console
kubectl apply -f examples/provider-default-gcp.yaml
```

### App Dev/Ops: Consume the infrastructure you need using kubectl

#### Provision a Kubernetes cluster using kubectl

1. Provision a `Cluster` resource (claim for `CompositeCluster`) provided by the platform `Configuration`:

```console
kubectl -n upbound-system apply -f examples/cluster.yaml
```

1. View status / details of the managed resources created for your claim:

```console
kubectl get managed
```

1. Check status of your claim:

```console
kubectl -n upbound-system get cluster
```

### Cleanup & Uninstall

#### Cleanup Resources

1. Delete `Cluster` claim:

```console
kubectl -n upbound-system delete -f examples/cluster.yaml
```

2. Verify all underlying resources have been cleanly deleted:

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

## Customize for your Organization

Create a `Repository` called `platform-ref-gcp` in your Upbound Cloud `Organization`.

Set these to match your settings:

```console
UPBOUND_ORG=acme
UPBOUND_ACCOUNT_EMAIL=me@acme.io
REPO=platform-ref-gcp
VERSION_TAG=v0.0.7
REGISTRY=registry.upbound.io
PLATFORM_CONFIG=${REGISTRY:+$REGISTRY/}${UPBOUND_ORG}/${REPO}:${VERSION_TAG}
```

Clone the GitHub repo.

```console
git clone https://github.com/upbound/platform-ref-gcp.git
cd platform-ref-gcp
```

Login to your container registry.

```console
docker login ${REGISTRY} -u ${UPBOUND_ACCOUNT_EMAIL}
```

Build package.

```console
up xpkg build --name package.xpkg --ignore ".github/*,.github/*/*,examples/*,hack/*"
```

Push package to registry.

```console
up xpkg push ${PLATFORM_CONFIG} -f package.xpkg
```

Install package into an Upbound `Control Plane` instance.

```console
kubectl crossplane install configuration ${PLATFORM_CONFIG}
```

The cloud service primitives that can be used in a `Composition` today are
listed in the Crossplane provider docs:

* [Upbound GCP Official Provider](https://marketplace.upbound.io/providers/upbound/provider-gcp)

To learn more see [Configuration
Packages](https://crossplane.io/docs/v0.14/getting-started/package-infrastructure.html).

## Learn More

If you're interested in building your own reference platform for your company,
we'd love to hear from you and chat. You can setup some time with us at
info@upbound.io.

For Crossplane questions, drop by [slack.crossplane.io](https://slack.crossplane.io), and say hi!
