# Snorkel Flow

Snorkel Flow is the data-centric AI platform for automated data labeling, integrated model training and analysis, and enhanced domain expert collaboration

For more information, visit our [official website](snorkel.ai)

# Architecture
![deployment](https://github.com/snorkel-ai/snorkel-flow-gcp-marketplace/assets/32317817/df430c33-089d-49e1-8b8d-b90991c5c071)

# Installation

## Quick install with Google Cloud Marketplace

Get up and running with a few clicks! Install Snorkel Flow on your Google Kubernetes Enginer cluster using Google Cloud Marketplace. Follow the [on-screen instructions](TODO:Add Listing).

## Command line instructions
### Set up command-line tools

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

### Install the Application resource definition

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

### Clone this repo

Clone this repo and the associated tools repo.

```shell
git clone --recursive https://github.com/snorkel-ai/snorkel-flow-gcp-marketplace.git
```
### Install the Application

Navigate to the `chart/snorkelflow` directory:

```shell
cd chart/snorkelflow
```

### Configure the app with environment variables

Choose an instance name and
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
for the app. In most cases, you can use the `snorkelflow` namespace.

```shell
export APP_INSTANCE_NAME=snorkel-flow
export NAMESPACE=snorkelflow
```

### Create namespace in your Kubernetes cluster

If you use a different namespace than `snorkelflow`, run the command below to create a new namespace:

```shell
kubectl create namespace "$NAMESPACE"
```

### Install the Helm Charts

Use `helm install` to install Snorkel Flow from the helm charts.
Note that you will need to insert the correct image paths in the `values.yaml`
file, or set them on the command line as shown here.

```shell
cd chart/snorkelflow

helm install --generate-name \
  --set image.imageNames.postgres="gcr.io/snorkelai-public/snorkelflow/postgres:0.76.11" \
  --set image.imageNames.engine="gcr.io/snorkelai-public/snorkelflow/engine:0.76.11" \
  --set image.imageNames.envoy="gcr.io/snorkelai-public/snorkelflow/envoy:0.76.11" \
  --set image.imageNames.flowUi="gcr.io/snorkelai-public/snorkelflow/flow-ui:0.76.11" \
  --set image.imageNames.grafana="gcr.io/snorkelai-public/snorkelflow/grafana:0.76.11" \
  --set image.imageNames.influxdb="gcr.io/snorkelai-public/snorkelflow/influxdb:0.76.11" \
  --set image.imageNames.minio="gcr.io/snorkelai-public/snorkelflow/minio:0.76.11" \
  --set image.imageNames.modelRegistry="gcr.io/snorkelai-public/snorkelflow/model-registry:0.76.11" \
  --set image.imageNames.notebook="gcr.io/snorkelai-public/snorkelflow/notebook:0.76.11" \
  --set image.imageNames.redis="gcr.io/snorkelai-public/snorkelflow/redis:0.76.11" \
  --set image.imageNames.secretsGenerator="gcr.io/snorkelai-public/snorkelflow/secrets-generator:0.76.11" \
  --set image.imageNames.singleuserNotebook="gcr.io/snorkelai-public/snorkelflow/singleuser-notebook:0.76.11" \
  --set image.imageNames.studio="gcr.io/snorkelai-public/snorkelflow/studio-api:0.76.11" \
  --set image.imageNames.telegraf="gcr.io/snorkelai-public/snorkelflow/telegraf:0.76.11" \
  --set image.imageNames.tdm="gcr.io/snorkelai-public/snorkelflow/tdm-api:0.76.11" \
  ./ 
```

The image versions here are just provided as a sample.

### View the app in the Google Cloud Console

To get the Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view your app, open the URL in your browser.

# Access Snorkel Flow

To access the Snorkel Flow UI, you can either expose a public service endpoint or keep it private, but connect
from your local environment with `kubectl port-forward`.

## Exposing the Snorkel Flow service publicly

Snorkel Flow will automatically create ingress objects of type `gce`,
which will by default create an external load balancer. We also offer the option to specify a custom domain within the deployer schema.

In order to find the public IP, run the following command:

```shell
kubectl get ingresses --namespace {{ .Release.Namespace }}
```

The external address will be the address for `snorkelflow-ingress`
Keep in mind that that the provisioning process for the load balancer can take a while.

## Forward Snorkel Flow port in local environment

You can use port forwarding feature of `kubectl` to forward Snorkel Flow's port to your local machine. Run the following command in background:

```shell
kubectl get pods --namespace {{ .Release.Namespace }}
```

Find the `flow-ui` pod, and then execute the following command:

```shell
kubectl port-forward --namespace {{ .Release.Namespace }} flow-ui-{pod name} 5000
```
Replace the exact name of the `flow-ui` pod with the actual name of the pod listed.

Now you can access Snorkel Flow with `http://localhost:5000/`.

## Login to Snorkel Flow
Upload the license provided to you and follow the instructions on screen to create user accounts.

# Scaling
This installation of Snorkel Flow is not meant to be scaled up.

# Upgrading the app
The image references in each pod can be set to the latest tag.

# Backup and Restore
The Snorkel Flow data volumes are stored on `PersistentVolume` objects, in order to view these, run

```shell
kubectl get pv --namespace {{ .Release.Namespace }}
```

The `snorkelflow-data` volume is created by the Google Filestore CSI driver, and it points to an automatically
provisioned NFS

The other data volumes are created by the CSI driver.

## snorkelflow-data

This can be done through both the Google Cloud Platform Console or through the CLI. 

[Reference](https://cloud.google.com/sdk/gcloud/reference/filestore/backups/create)

To backup through the CLI, first find the instance name of the volume. This will be in the form of
`pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`, the file share name will be the the string at the end of the 
`VolumeHandle` when executing 

```shell
kubectl describe pv pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx --namespace {{ .Release.Namespace }}
```

For example, the `VolumeHandle` could be `modeInstance/us-central1-c/pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/vol1`
where the file share name will then be `vol1`

Next, execute the following command, substituting in your specific file share and instance id.

```shell
gcloud filestore backups create my-backup --file-share=vol1 --instance=pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx--instance-zone=us-central1-c --region=us-central1 --description="Backup"
```

Now in the [Backups](https://console.cloud.google.com/filestore/backups?project=snorkelai-public) console, we can restore this backup to the same Filestore instance.

## Other volumes

As the other volumes are provisioned by the Compute Engine persistent disk CSI driver, follow these [instructions](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/volume-snapshots#v1) in order to use Volume Snapshots to backup and restore the other persistent volumes.


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

Alternatively, if the namespace is no longer required, you can delete the namespace itself

```shell
kubectl delete namespace ${NAMESPACE}
```

### Delete the GKE cluster

Optionally, if you don't need the deployed application or the GKE cluster,
delete the cluster using this command:

```shell
gcloud container clusters delete "${CLUSTER}" --zone "${ZONE}"
```

