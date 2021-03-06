+++
title = "Deploy using CLI"
description = "Instructions for using the CLI to deploy Kubeflow on Google Cloud Platform (GCP)"
weight = 4
+++

This guide describes how to use the `kfctl` command line interface (CLI) to
deploy Kubeflow on GCP. The command line deployment gives you more control over
the deployment process and configuration than you get if you use the deployment 
UI. If you're looking for a simpler deployment procedure, see how to deploy 
Kubeflow [using the deployment UI](/docs/gke/deploy/deploy-ui).

## Before you start

Before installing Kubeflow on the command line:

1. Ensure you have installed the following tools:
  
  * [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
  * [gcloud](https://cloud.google.com/sdk/). If you already have `gcloud`
    installed, run `gcloud components update` to
     get the latest version of all your installed Cloud SDK components.

1. If you're using
  [Cloud Shell](https://cloud.google.com/shell/), enable 
  [boost mode](https://cloud.google.com/shell/docs/features#boost_mode).

1. Make sure that your GCP project meets the minimum requirements
  described in the [project setup guide](/docs/gke/deploy/project-setup/).

1. If you want to use [Cloud Identity-Aware Proxy (Cloud 
  IAP)](https://cloud.google.com/iap/docs/) for access control, follow the guide
  to [setting up OAuth credentials](/docs/gke/deploy/oauth-setup/). 
  Cloud IAP is recommended for production deployments or deployments with 
  access to sensitive data. Alternatively, you can use basic authentication 
  with a username and password.

<a id="prepare-environment"></a>
## Prepare your environment

Follow these steps to download the kfctl binary for the Kubeflow CLI and set
some handy environment variables:

1. Download the kfctl {{% kf-latest-version %}} release from the
  [Kubeflow releases 
  page](https://github.com/kubeflow/kubeflow/releases/tag/{{% kf-latest-version %}}).

1. Unpack the tar ball:

    ```
    tar -xvf kfctl_{{% kf-latest-version %}}_<platform>.tar.gz
    ```

1. Create user credentials. You only need to run this command once:
   
    ```
    gcloud auth application-default login
    ```

1. Create environment variables to make the deployment process easier:

    ```
    # Set your GCP project ID and the zone where you want to create 
    # the Kubeflow deployment:
    export PROJECT=<your GCP project ID>
    gcloud config set project ${PROJECT}
    export ZONE=<your GCP zone>
    gcloud config set compute/zone ${ZONE}

    # Use the following kfctl configuration file for authentication with 
    # Cloud IAP (recommended):
    export CONFIG_URI="{{% config-uri-gcp-iap %}}"

    # If using Cloud IAP for authentication, create environment variables
    # from the OAuth client ID and secret that you obtained earlier:
    export CLIENT_ID=<CLIENT_ID from OAuth page>
    export CLIENT_SECRET=<CLIENT_SECRET from OAuth page>

    # Alternatively, use the following kfctl configuration if you want to use 
    # basic authentication:
    export CONFIG_URI="{{% config-uri-gcp-basic-auth %}}"

    # If using basic authentication, create environment variables
    # for username and password:
    export KUBEFLOW_USERNAME=<your username>
    export KUBEFLOW_PASSWORD=<your password>

    # Set KF_NAME to the name of your Kubeflow deployment. You also use this
    # value as directory name when creating your configuration directory. 
    # See the detailed description in the text below this code snippet.
    # For example, your deployment name can be 'my-kubeflow' or 'kf-test'.
    export KF_NAME=<your choice of name for the Kubeflow deployment>

    # Set the path to the base directory where you want to store one or more 
    # Kubeflow deployments. For example, /opt/.
    # Then set the Kubeflow application directory for this deployment.
    export BASE_DIR=<path to a base directory>
    export KF_DIR=${BASE_DIR}/${KF_NAME}

    # The following command is optional. It adds the kfctl binary to your path.
    # If you don't add kfctl to your path, you must use the full path
    # each time you run kfctl.
    export PATH=$PATH:<path to your kfctl file>
    ```

Notes:

* **${PROJECT}** - The project ID of the GCP project where you want Kubeflow 
  deployed.
* **${ZONE}** - The GCP zone where you want to create the Kubeflow deployment.
  You can see a list of zones in the 
  [Compute Engine documentation](https://cloud.google.com/compute/docs/regions-zones/#available).
  If you plan to use accelerators, you must choose a zone that supports the
  type you want. See the guide to 
  [customizing your Kubeflow deployment](/docs/gke/customizing-gke/#gpu-config).
* **${CONFIG_URI}** - The GitHub address of the configuration YAML file that
  you want to use to deploy Kubeflow. For GCP deployments, the following
  configurations are available:

  * `{{% config-uri-gcp-iap %}}` 
  * `{{% config-uri-gcp-basic-auth %}}`

    When you run `kfctl apply` or `kfctl build` (see the next step), kfctl creates
    a local version of the configuration YAML file which you can further
    customize if necessary.

* **${KF_NAME}** - The name of your Kubeflow deployment.
  If you want a custom deployment name, specify that name here.
  For example,  `my-kubeflow` or `kf-test`.
  The value of KF_NAME must consist of lower case alphanumeric characters or
  '-', and must start and end with an alphanumeric character.
  The value of this variable cannot be greater than 25 characters. It must
  contain just a name, not a directory path.
  You also use this value as directory name when creating the directory where 
  your Kubeflow  configurations are stored, that is, the Kubeflow application 
  directory. 
  
* **${KF_DIR}** - The full path to your Kubeflow application directory.

<a id="set-up-and-deploy"></a>
## Set up and deploy Kubeflow

To set up and deploy Kubeflow using the **default settings**,
run the `kfctl apply` command:

```
mkdir -p ${KF_DIR}
cd ${KF_DIR}
kfctl apply -V -f ${CONFIG_URI}
```

## Alternatively, set up your configuration for later deployment

If you want to customize your configuration before deploying Kubeflow, you can 
set up your configuration files first, then edit the configuration, then
deploy Kubeflow:

1. Run the `kfctl build` command to set up your configuration:

  ```
  mkdir -p ${KF_DIR}
  cd ${KF_DIR}
  kfctl build -V -f ${CONFIG_URI}
  ```

1. Edit the configuration files, as described in the guide to
  [customizing your Kubeflow deployment](/docs/gke/customizing-gke/).

1. Set an environment variable for your local configuration file:

  ```
  export CONFIG_FILE=${KF_DIR}/{{% config-file-gcp-iap %}}
  ```

    Or:

  ```
  export CONFIG_FILE=${KF_DIR}/{{% config-file-gcp-basic-auth %}}
  ```

1. Run the `kfctl apply` command to deploy Kubeflow:

  ```
  kfctl apply -V -f ${CONFIG_FILE}
  ```

## Check your deployment

Follow these steps to verify the deployment:

1. The deployment process creates a separate deployment for your data storage. 
   After running `kfctl apply` you should notice two new 
   [deployments](https://console.cloud.google.com/dm/deployments):
   * **{KF_NAME}-storage**: This deployment has persistent volumes for your
     pipelines.
   * **{KF_NAME}**: This deployment has all the components of Kubeflow, including 
     a [GKE cluster](https://console.cloud.google.com/kubernetes/list) 
     named **${KF_NAME}** with Kubeflow installed.

1. When the deployment finishes, check the resources installed in the namespace
   `kubeflow` in your new cluster.  To do this from the command line, first set 
   your `kubectl` credentials to point to the new cluster:

    ```
    gcloud container clusters get-credentials ${KF_NAME} --zone ${ZONE} --project ${PROJECT}
    ```

    Then see what's installed in the `kubeflow` namespace of your GKE cluster:

    ```
    kubectl -n kubeflow get all
    ```

## Access the Kubeflow user interface (UI)

Follow these steps to access the Kubeflow central dashboard:

1. Enter the following URI into your browser address bar. It can take 20
  minutes for the URI to become available:

  ```
  https://<KF_NAME>.endpoints.<project-id>.cloud.goog/
  ```

    You can run the following command to get the URI for your deployment:

  ```
  kubectl -n istio-system get ingress
  NAME            HOSTS                                                      ADDRESS         PORTS   AGE
  envoy-ingress   your-kubeflow-name.endpoints.your-gcp-project.cloud.goog   34.102.232.34   80      5d13h
  ```

    The following command sets an environment variable named `HOST` to the URI:

  ```
  export HOST=$(kubectl -n istio-system get ingress envoy-ingress -o=jsonpath={.spec.rules[0].host})
  ```

1. Follow the instructions on the UI to create a namespace. See the guide to 
  [creation of profiles](/docs/other-guides/multi-user-overview/#automatic-creation-of-profiles).

Notes:

* It can take 20 minutes for the URI to become available.
  Kubeflow needs to provision a signed SSL certificate and register a DNS 
  name.
* If you own or manage the domain or a subdomain with 
  [Cloud DNS](https://cloud.google.com/dns/docs/)
  then you can configure this process to be much faster.
  See [kubeflow/kubeflow#731](https://github.com/kubeflow/kubeflow/issues/731).

## Understanding the deployment process

This section gives you more details about the kfctl configuration and 
deployment process, so that you can customize your Kubeflow deployment if
necessary.

### kfctl process and configuration

The kfctl deployment process includes the following commands:

* `kfctl build` - (Optional) Creates configuration files defining the various 
  resources in your deployment. You only need to run `kfctl build` if you want 
  to edit the resources before running `kfctl apply`. See the guide to
  [customizing your Kubeflow deployment](/docs/gke/customizing-gke/).
* `kfctl apply` - Creates or updates the resources.
* `kfctl delete` - Deletes the resources.

The kfctl deployment process applies default values to certain properties
as follows:

* **Email address:** kfctl attempts to fetch your email address from your
  Cloud SDK configuration. You can run `gcloud config list` to see the default 
  email address, which the command output lists as the **account**.
  If kfctl can't find a valid email address, you must use the
  flag `--email <your email address>` to pass a valid email address. This email 
  address becomes an administrator in the configuration of your Kubeflow 
  deployment.

* **GCP project ID:** kfctl attempts to fetch your project ID from your
  Cloud SDK configuration. You can run `gcloud config list` to see your 
  active project ID.

* **GCP zone:** kfctl attempts to fetch the zone from your Cloud SDK
  configuration. You can run `gcloud config list` to see your active zone.

* **Kubeflow deployment name:** kfctl defaults to the name of the directory
  where you run the `kfctl build` or `kfctl apply` command.

You can also explicitly set the following values in your `${CONFIG_FILE}`
configuration file:

* Kubeflow deployment name
* GCP project
* GCP zone
* Email address


The following snippet shows you how to set values in the configuration file
using [yq](https://github.com/mikefarah/yq/releases):

```
yq w -i ${CONFIG_FILE} spec.plugins[0].spec.project ${PROJECT}
yq w -i ${CONFIG_FILE} spec.plugins[0].spec.zone ${ZONE}
yq w -i ${CONFIG_FILE} metadata.name ${KF_NAME}
```

### Application layout

Your Kubeflow application directory **${KF_DIR}** contains the following files and 
directories:

* **${CONFIG_FILE}** is a YAML file that defines configurations related to your 
  Kubeflow deployment.

  * This file is a copy of the GitHub-based configuration YAML file that
    you used when deploying Kubeflow: 
      * either `{{% config-uri-gcp-iap %}}`
      * or `{{% config-uri-gcp-basic-auth %}}`.
  * When you run `kfctl apply` or `kfctl build`, kfctl creates
    a local version of the configuration file, **${CONFIG_FILE}**,
    which you can further customize if necessary.

* **gcp_config** is a directory that contains 
  [Deployment Manager configuration files](https://cloud.google.com/deployment-manager/docs/configuration/) 
  defining your GCP infrastructure.

  * The directory is created when you run `kfctl build` or `kfctl apply`.
  * You can modify these configurations to customize your GCP infrastructure.
    After modifying a configuration, run `kfctl apply` again.

* **kustomize** is a directory that contains the kustomize packages for Kubeflow 
  applications. See 
  [how Kubeflow uses kustomize](/docs/other-guides/kustomize/).

  * The directory is created when you run `kfctl build` or `kfctl apply`.
  * You can customize the Kubernetes resources by modifying the manifests and 
    running `kfctl apply` again.

We recommend that you check in the contents of your **${KF_DIR}** directory
into source control.

### GCP service accounts

The kfctl deployment process creates three service accounts in your 
GCP project. These service accounts follow the [principle of least 
privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege). 
The service accounts are:

* `${KF_NAME}-admin` is used for some admin tasks like configuring the load 
  balancers. The principle is that this account is needed to deploy Kubeflow but 
  not needed to actually run jobs.
* `${KF_NAME}-user` is intended to be used by training jobs and models to access 
  GCP resources (Cloud Storage, BigQuery, etc.). This account has a much smaller 
  set of privileges compared to `admin`.
* `${KF_NAME}-vm` is used only for the virtual machine (VM) service account. This
  account has the minimal permissions needed to send metrics and logs to 
  [Stackdriver](https://cloud.google.com/stackdriver/).

## Next steps

* Run a full ML workflow on Kubeflow, using the
  [end-to-end MNIST tutorial](/docs/gke/gcp-e2e/) or the
  [GitHub issue summarization 
  example](https://github.com/kubeflow/examples/tree/master/github_issue_summarization).
* See how to [delete](/docs/gke/deploy/delete-cli/) your Kubeflow deployment 
  using the CLI.
* See how to [customize](/docs/gke/customizing-gke/) your Kubeflow 
  deployment.
* See how to [upgrade Kubeflow](/docs/upgrading/upgrade/) and how to 
  [upgrade or reinstall a Kubeflow Pipelines 
  deployment](/docs/pipelines/upgrade/).
* [Troubleshoot](/docs/gke/troubleshooting-gke/) any issues you may
  find.
