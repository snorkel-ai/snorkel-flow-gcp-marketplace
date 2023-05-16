# Snorkel Flow

Snorkel Flow is a data-centric AI platform for automated data labeling, integrated model training and analysis, and enhanced domain expert collaboration

For more information, visit our [official website](snorkel.ai)

## Architecture
(Diagram)

# Installation

## Quick install with Google Cloud Marketplace

Get up and running with a few clicks! Install Snorkel Flow on your Google Kubernetes Enginer cluster using Google Cloud Marketplace. Follow the [on-screen instructions](TODO:Add Listing).

## Command line instructions
### Prerequisites

#### Set up command-line tools

You'll need the following tools in your development environment. If you are
using Cloud Shell, `gcloud`, `kubectl`, Docker, and Git are installed in your
environment by default.

-   [gcloud](https://cloud.google.com/sdk/gcloud/)
-   [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
-   [docker](https://docs.docker.com/install/)
-   [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
-   [helm](https://helm.sh/)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

### Create a Google Kubernetes Engine cluster

Create a new cluster from the command line:

```shell
export CLUSTER=snorkelflow
export ZONE=us-west1-a

gcloud container clusters create "$CLUSTER" --zone "$ZONE"
```

Configure `kubectl` to connect to the new cluster:

```shell
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE"
```

### Set up Filestore CSI driver

Snorkel Flow requires that the [Filestore CSI Driver](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/filestore-csi-driver) is installed
on your cluster. To enable this driver on an existing cluster:

```shell
gcloud container clusters update "$CLUSTER" \
   --update-addons=GcpFilestoreCsiDriver=ENABLED
   --zone "$ZONE"
```

#### Install the Application resource definition

An Application resource is a collection of individual Kubernetes components,
such as Services, Deployments, and so on, that you can manage as a group.

To set up your cluster to understand Application resources, run the following
command:

```shell
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
```

You need to run this command once.

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. The source code can be found on
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

#### Clone this repo

Clone this repo and the associated tools repo.

```shell
git clone --recursive https://github.com/snorkel-ai/snorkel-flow-gcp-marketplace.git
```
### Install the Application

Navigate to the `chart/snorkelflow` directory:

```shell
cd chart/snorkelflow
```

#### Configure the app with environment variables

Choose an instance name and
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
for the app. In most cases, you can use the `default` namespace.

```shell
export APP_INSTANCE_NAME=snorkel-flow
export NAMESPACE=snorkelflow
```

Enable Stackdriver Metrics Exporter:

> **NOTE:** Your GCP project must have Stackdriver enabled. If you are using a
> non-GCP cluster, you cannot export metrics to Stackdriver.

By default, the application does not export metrics to Stackdriver. To enable
this option, change the value to `true`.

```shell
export METRICS_EXPORTER_ENABLED=false
```

#### Create namespace in your Kubernetes cluster

If you use a different namespace than `snorkelflow`, run the command below to create
a new namespace:

```shell
kubectl create namespace "$NAMESPACE"
```

#### Expand the manifest template

Use `helm install` to install from helm charts.
Navigate to the helm chart directory

```shell
cd chart/snorkelflow

helm install --values ./values.yaml ./ --generate-name
```

#### View the app in the Google Cloud Console

To get the Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view your app, open the URL in your browser.

# Access Snorkel Flow

Snorkel Flow can be accessed 

# Scaling
This installation of Snorkel Flow is not meant to be scaled up.

# Upgrading the app

# Uninstalling the Application

## Using the Google Cloud Platform Console

In the GCP Console, open [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).
From the list of applications, click **Snorkel Flow**.
On the Application Details page, click **Delete**.

## Using the command line
### Delete the resources

> **NOTE:** We recommend that you use a `kubectl` version that is the same as
> the version of your cluster. Using the same versions of `kubectl` and the
> cluster helps avoid unforeseen issues.

```shell
kubectl delete daemonset,service,configmap,application \
  --namespace ${NAMESPACE} \
  --selector app.kubernetes.io/name=${APP_INSTANCE_NAME}
```

### Delete the GKE cluster

Optionally, if you don't need the deployed application or the GKE cluster,
delete the cluster using this command:

```shell
gcloud container clusters delete "${CLUSTER}" --zone "${ZONE}"
```

